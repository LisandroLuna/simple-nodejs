pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "maguilaes/simple-nodejs"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/maguilaes/simple-nodejs.git'
            }
        }

        stage('Build Docker Image') {
            steps { 
                script {
                    // Build with branch-buildnumber tag for consistency
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} ."
                } 
            }
        }      
        
        stage('Test') {
            steps {
                script {
                    // Use the same tag we built with
                    sh "docker run --rm ${DOCKER_IMAGE_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} npm test"
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    // Verify credentials exist in Jenkins with this ID
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials', // Must match Jenkins credential ID
                        passwordVariable: 'DOCKER_HUB_PASSWORD',
                        usernameVariable: 'DOCKER_HUB_USERNAME'
                    ]) {
                        // Removed sudo for consistency
                        sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                        sh "docker push ${DOCKER_IMAGE_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                        
                        if (env.BRANCH_NAME == 'main') {
                            // Tag from existing image
                            sh "docker tag ${DOCKER_IMAGE_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} ${DOCKER_IMAGE_NAME}:latest"
                            sh "docker push ${DOCKER_IMAGE_NAME}:latest"
                        }
                    }
                }
            }
        }
    }
        
    post {
        always {
            script {
                try {
                    // Clean up all tags we created
                    sh "docker rmi ${DOCKER_IMAGE_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} || true"
                    if (env.BRANCH_NAME == 'main') {
                        sh "docker rmi ${DOCKER_IMAGE_NAME}:latest || true"
                    }
                } catch (Exception e) {
                    echo 'Failed to remove Docker images. Continuing...'
                }
            }
        }
    }
}