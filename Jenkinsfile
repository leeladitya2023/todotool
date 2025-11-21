pipeline {
    agent any
    
    environment {
        // Docker image name and tag
        DOCKER_IMAGE = 'todo-reduxtoolkit'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_REGISTRY = 'your-registry.com' // Update with your Docker registry
        DOCKER_CREDENTIALS = 'docker-credentials' // Update with your Jenkins Docker credentials ID
        
        // Application configuration
        APP_PORT = '3000'
        CONTAINER_NAME = 'todo-reduxtoolkit-app'
        
        // Deployment
        DEPLOY_HOST = 'your-deployment-host.com' // Update with your deployment server
        DEPLOY_USER = 'deploy' // Update with your deployment user
        DEPLOY_SSH_CREDENTIALS = 'ssh-credentials' // Update with your SSH credentials ID
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "Checking out code from repository..."
                    checkout scm
                }
            }
        }
        
        stage('Lint') {
            steps {
                script {
                    echo "Running ESLint..."
                    sh '''
                        npm ci
                        npm run lint || true
                    '''
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    echo "Running tests..."
                    sh '''
                        npm ci
                        npm test -- --coverage --watchAll=false || true
                    '''
                }
            }
            post {
                always {
                    // Publish test results
                    publishTestResults testResultsPattern: 'test-results.xml'
                    // Publish coverage reports if available
                    publishHTML([
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
        
        stage('Build Application') {
            steps {
                script {
                    echo "Building React application..."
                    sh '''
                        npm ci
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
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                            docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                            docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                            
                            # Login to Docker registry
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin ${DOCKER_REGISTRY}
                            
                            # Tag for registry
                            docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker tag ${DOCKER_IMAGE}:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                        '''
                    }
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    echo "Pushing Docker image to registry..."
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                        '''
                    }
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    echo "Scanning Docker image for vulnerabilities..."
                    // Install Trivy if not available
                    sh '''
                        which trivy || (wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | apt-key add - && \
                        echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/trivy.list && \
                        apt-get update && apt-get install -y trivy)
                    '''
                    sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKER_IMAGE}:${DOCKER_TAG} || true"
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop' // Deploy to staging from develop branch
            }
            steps {
                script {
                    echo "Deploying to staging environment..."
                    sshagent([env.DEPLOY_SSH_CREDENTIALS]) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} \
                                "docker pull ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG} && \
                                 docker stop ${CONTAINER_NAME}-staging || true && \
                                 docker rm ${CONTAINER_NAME}-staging || true && \
                                 docker run -d --name ${CONTAINER_NAME}-staging -p 3001:80 --restart unless-stopped ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                        '''
                    }
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main' // Deploy to production from main branch
            }
            steps {
                script {
                    echo "Deploying to production environment..."
                    sshagent([env.DEPLOY_SSH_CREDENTIALS]) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} \
                                "docker pull ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG} && \
                                 docker stop ${CONTAINER_NAME} || true && \
                                 docker rm ${CONTAINER_NAME} || true && \
                                 docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:80 --restart unless-stopped ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
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
                // Clean up old Docker images locally
                sh '''
                    docker image prune -f
                    docker system prune -f
                '''
            }
        }
        success {
            echo "Pipeline completed successfully! ✓"
            // Send notification (configure as needed)
            // emailext (
            //     subject: "Pipeline Success: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
            //     body: "Build ${env.BUILD_NUMBER} completed successfully.",
            //     to: "team@example.com"
            // )
        }
        failure {
            echo "Pipeline failed! ✗"
            // Send notification (configure as needed)
            // emailext (
            //     subject: "Pipeline Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
            //     body: "Build ${env.BUILD_NUMBER} failed. Please check the logs.",
            //     to: "team@example.com"
            // )
        }
        cleanup {
            // Clean workspace
            cleanWs()
        }
    }
}

