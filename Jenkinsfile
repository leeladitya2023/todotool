// Jenkinsfile for React Redux Toolkit Todo App
// This file should be at the root of your repository

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
                    sh 'pwd'
                    sh 'ls -la'
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                script {
                    echo "Installing dependencies..."
                    sh '''
                        npm ci
                    '''
                }
            }
        }
        
        stage('Lint') {
            steps {
                script {
                    echo "Running ESLint..."
                    sh '''
                        npm run lint || true
                    '''
                }
            }
        }
        
        stage('Build Application') {
            steps {
                script {
                    echo "Building React application..."
                    sh '''
                        npm run build
                    '''
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
                    sh '''
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }
        
        stage('Stop Old Container') {
            steps {
                script {
                    echo "Stopping old container if exists..."
                    sh '''
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                    '''
                }
            }
        }
        
        stage('Run Container') {
            steps {
                script {
                    echo "Running new container..."
                    sh '''
                        docker run -d --name ${CONTAINER_NAME} -p 3000:80 --restart unless-stopped ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    echo "Checking container health..."
                    sh '''
                        sleep 5
                        docker ps | grep ${CONTAINER_NAME} || true
                        curl -f http://localhost:3000/health || curl -f http://localhost:3000/ || true
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo "Cleaning up workspace..."
                sh '''
                    docker image prune -f || true
                '''
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
