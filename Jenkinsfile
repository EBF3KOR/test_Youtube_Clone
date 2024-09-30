pipeline {
    agent any

    environment {
        NODE_VERSION = '16.14.0'
        PORT = '3000'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup') {
            steps {
                bat "nvm use ${NODE_VERSION} || nvm install ${NODE_VERSION}"
                bat 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                bat 'npx eslint src'
            }
        }

        stage('Test') {
            steps {
                bat 'npm test -- --watchAll=false'
            }
        }

        stage('Build') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Deploy Locally') {
            steps {
                script {
                    // Check if a process is already running on PORT 3000
                    def isPortInUse = bat(script: "netstat -ano | findstr :${PORT}", returnStatus: true) == 0
                    if (isPortInUse) {
                        // Kill the process using PORT 3000
                        def pid = bat(script: "netstat -ano | findstr :${PORT} | findstr LISTENING | awk '{print \$5}'", returnStdout: true).trim()
                        if (pid) {
                            bat "taskkill /PID ${pid} /F"
                        }
                    }

                    // Start the application
                    bat "start /B npm start > app.log 2>&1"
                    
                    // Wait for the application to start
                    script {
                        sleep(time: 5, unit: 'SECONDS') // wait for a bit
                        def checkUrl = "curl -f http://localhost:${PORT}"
                        retry(3) {
                            bat checkUrl
                        }
                    }
                    echo "Application started successfully on port ${PORT}"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                bat "curl -f http://localhost:${PORT}"
                echo "Application is accessible at http://localhost:${PORT}"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
