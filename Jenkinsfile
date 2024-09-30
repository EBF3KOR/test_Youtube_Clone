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
                sh "nvm use ${NODE_VERSION} || nvm install ${NODE_VERSION}"
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npx eslint src'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test -- --watchAll=false'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy Locally') {
            steps {
                script {
                    // Check if a process is already running on PORT 3000
                    def isPortInUse = sh(script: "lsof -i:${PORT}", returnStatus: true) == 0
                    if (isPortInUse) {
                        // Kill the process using PORT 3000
                        sh "lsof -ti:${PORT} | xargs kill -9"
                    }
                    
                    // Start the application
                    sh "nohup npm start > app.log 2>&1 &"
                    
                    // Wait for the application to start
                    sh "until lsof -i:${PORT} > /dev/null 2>&1; do sleep 1; done"
                    echo "Application started successfully on port ${PORT}"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "curl -f http://localhost:${PORT}"
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
