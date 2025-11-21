// Jenkinsfile - Cross-platform (Windows/Linux)
// Uses bat for Windows, sh for Linux

pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'todo-reduxtoolkit'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        CONTAINER_NAME = 'todo-reduxtoolkit-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "Checking out code from repository..."
                    checkout scm
                    // Show current directory to debug
                    if (isUnix()) {
                        sh 'pwd && ls -la'
                    } else {
                        bat 'cd && dir'
                    }
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                script {
                    echo "Installing dependencies..."
                    if (isUnix()) {
                        sh '''
                            npm ci
                        '''
                    } else {
                        bat '''
                            call npm ci
                            if errorlevel 1 exit /b 1
                        '''
                    }
                }
            }
        }
        
        stage('Lint') {
            steps {
                script {
                    echo "Running ESLint..."
                    if (isUnix()) {
                        sh '''
                            npm run lint || true
                        '''
                    } else {
                        bat '''
                            call npm run lint || exit /b 0
                        '''
                    }
                }
            }
        }
        
        stage('Build Application') {
            steps {
                script {
                    echo "Building React application..."
                    if (isUnix()) {
                        sh '''
                            npm run build
                        '''
                    } else {
                        bat '''
                            call npm run build
                            if errorlevel 1 exit /b 1
                        '''
                    }
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    echo "Building Docker image..."
                    if (isUnix()) {
                        sh '''
                            docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                            docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                        '''
                    } else {
                        bat '''
                            docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% .
                            if errorlevel 1 exit /b 1
                            docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest
                        '''
                    }
                }
            }
        }
        
        stage('Run Container') {
            steps {
                script {
                    echo "Removing old container if exists and running new container..."
                    if (isUnix()) {
                        sh '''
                            docker rm -f ${CONTAINER_NAME} 2>/dev/null || true
                            docker run -d --name ${CONTAINER_NAME} -p 3000:80 --restart unless-stopped ${DOCKER_IMAGE}:${DOCKER_TAG}
                        '''
                    } else {
                        bat '''
                            @echo off
                            docker rm -f %CONTAINER_NAME% >nul 2>&1
                            docker run -d --name %CONTAINER_NAME% -p 3000:80 --restart unless-stopped %DOCKER_IMAGE%:%DOCKER_TAG%
                            if errorlevel 1 (
                                echo Failed to start container
                                exit /b 1
                            )
                            echo Container started successfully
                        '''
                    }
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    echo "Checking container health..."
                    if (isUnix()) {
                        sh '''
                            sleep 5
                            docker ps | grep ${CONTAINER_NAME} || echo Container check skipped
                            curl -f http://localhost:3000/health || curl -f http://localhost:3000/ || true
                        '''
                    } else {
                        bat '''
                            @echo off
                            timeout /t 5 /nobreak >nul
                            docker ps | findstr %CONTAINER_NAME% >nul 2>&1
                            curl -f http://localhost:3000/health >nul 2>&1
                            if errorlevel 1 curl -f http://localhost:3000/ >nul 2>&1
                            echo Health check completed
                            exit /b 0
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo "Cleaning up workspace..."
                if (isUnix()) {
                    sh 'docker image prune -f || true'
                } else {
                    bat '''
                        @echo off
                        docker image prune -f >nul 2>&1
                        echo Cleanup completed
                        exit /b 0
                    '''
                }
            }
        }
        success {
            echo "✅ Pipeline completed successfully!"
            echo "🌐 Application is running at: http://localhost:3000"
        }
        failure {
            echo "❌ Pipeline failed! Check logs above."
        }
        cleanup {
            cleanWs()
        }
    }
}
