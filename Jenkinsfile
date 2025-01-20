pipeline {
    agent any

    environment {
        IMAGE_NAME = 'test1'
        IMAGE_TAG = 'latest'
        AWS_REGION = credentials('AWS_REGION')

    }

    stages {
        stage('Clean Workspace') {
            steps {
                script {
                    deleteDir()
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
                        string(credentialsId: 'ECR_REPO', variable: 'ECR_REPO')
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
                    withAWS(credentials: 'aws-credentials-id', region: "${AWS_REGION}") {
                        docker.withRegistry("https://${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com", 'ecr:aws-credentials-id') {
                            sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
                        }
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
