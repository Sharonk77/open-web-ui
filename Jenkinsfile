pipeline {
    agent any

    environment {
        IMAGE_NAME = credentials('IMAGE_NAME')
//         IMAGE_TAG = credentials('IMAGE_TAG')
        IMAGE_TAG = 'sharonimagelala15'
        AWS_REGION = credentials('AWS_REGION')
        AWS_ACCOUNT_ID = credentials('AWS_ACCOUNT_ID')
        ECR_REPO = credentials('ECR_REPO')
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
                    git checkout v0.5.3
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
                            echo "Current directory:" $(pwd)
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
                        string(credentialsId: 'ECR_REPO', variable: 'ECR_REPO')
                    ]) {
                        withAWS(credentials: 'aws-jenkins-cred', region: 'us-east-1') {
                            docker.withRegistry("https://${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com", "ecr:us-east-1:aws-jenkins-cred") {
                                sh """
                                docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/${IMAGE_NAME}:${IMAGE_TAG}
                                echo "pushed to ECR..."

                                """
                            }
                        }
                    }
                }
            }
        }

        stage('deploying openwebui') {
            steps {
                script {

                    sh '''
                    echo "Cloning repository..."
                    git clone https://github.com/Sharonk77/open-web-ui/
                    cd open-web-ui
                    echo "Current directory:" $(pwd)
                    echo "executing sed command"
                    kubectl set image deployment/openwebui openwebui=${IMAGE_NAME}:${IMAGE_TAG}


                    '''
                }
            }
        }
    }


    post {
        success {
            echo "success"
        }
        failure {
            echo "Failed"
        }
    }
}
