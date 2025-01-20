pipeline {
    agent any

    environment {
        IMAGE_NAME = 'open-web-ui/openweb'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                script {
                    deleteDir()
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'AWS_REGION', variable: 'AWS_REGION'),
                        string(credentialsId: 'ECR_REPO', variable: 'ECR_REPO')
                    ]) {
                        sh '''
                        echo "Logging in to Amazon ECR..."
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $ECR_REPO
                        '''
                    }
                }
            }
        }

        stage('Clone and Build Docker Image') {
            steps {
                sh '''
                echo "Cloning repository..."
                git clone https://github.com/open-webui/open-webui
                cd open-webui
                echo "Building Docker image..."
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'ecr-repo', variable: 'ECR_REPO')
                    ]) {
                        sh '''
                        echo "Tagging Docker image..."
                        docker tag $IMAGE_NAME:$IMAGE_TAG $ECR_REPO:$IMAGE_TAG
                        '''
                    }
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'ecr-repo', variable: 'ECR_REPO')
                    ]) {
                        sh '''
                        echo "Pushing Docker image to Amazon ECR..."
                        docker push $ECR_REPO:$IMAGE_TAG
                        '''
                    }
                }
            }
        }

        stage('Cleanup Docker Images') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'ecr-repo', variable: 'ECR_REPO')
                    ]) {
                        sh '''
                        echo "Cleaning up local Docker images..."
                        docker rmi $IMAGE_NAME:$IMAGE_TAG $ECR_REPO:$IMAGE_TAG
                        docker logout $ECR_REPO
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Docker image successfully pushed to AWS ECR!"
        }
        failure {
            echo "Failed to push Docker image to AWS ECR."
        }
    }
}
