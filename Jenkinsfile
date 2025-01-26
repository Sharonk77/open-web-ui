pipeline {
    agent any

    environment {
        IMAGE_NAME = credentials('IMAGE_NAME')
        IMAGE_TAG = credentials('IMAGE_TAG')
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
                    echo "Building Docker image..."
                    docker build -t $IMAGE_NAME .
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
                                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/${IMAGE_NAME}:${IMAGE_TAG}"
                            }
                        }
                    }
                }
            }
        }

//         stage('Replace Variables in Deployment File') {
//             steps {
//                 script {
//                     def deploymentFile = 'deployment.yaml'
//
//                     sh """
//                         sed -i -e "s#{{AWS_ACCOUNT_ID}}#${AWS_ACCOUNT_ID}#g" \
//                                -e "s#{{AWS_REGION}}#${AWS_REGION}#g"  \
//                                -e "s#{{IMAGE_NAME}}#${IMAGE_NAME}#g" \
//                                -e "s#{{IMAGE_TAG}}#${IMAGE_TAG}#g" \
//                                open-web-ui-deployment.yaml
//                     """
//                     echo "Deployment file updated with credentials."
//                 }
//             }
//         }

//         stage('Install AWS CLI if needed') {
//             steps {
//                 script {
//                     sh '''
//                         if ! command -v aws &> /dev/null
//                         then
//                             echo "AWS CLI not found, installing..."
//                             curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
//                             unzip awscliv2.zip
//                             sudo ./aws/install
//                         else
//                             echo "AWS CLI already installed"
//                         fi
//                     '''
//                 }
//             }
//         }

//         stage('fffff') {
//             steps {
//                 script {
//                     withCredentials([
//                         string(credentialsId: 'ECR_REPO', variable: 'ECR_REPO')
//                     ]) {
//                         withAWS(credentials: 'aws-jenkins-cred', region: 'us-east-1') {
//                             docker.withRegistry("https://${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com", "ecr:us-east-1:aws-jenkins-cred") {
//                                 sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/${IMAGE_NAME}:${IMAGE_TAG}"
//                                 sh '''
//                                     echo "Creating Kubernetes secret for ECR authentication..."
//                                     kubectl create secret docker-registry ecr-secret \
//                                         --docker-server=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com \
//                                         --docker-username=AWS \
//                                         --docker-password=\$(aws ecr get-login-password --region ${AWS_REGION}) \
//                                         --namespace default --dry-run=client -o yaml | kubectl apply -f -
//
//                                     echo "Deploying to Kubernetes..."
//                                     kubectl apply -f open-web-ui-deployment.yaml
//                                 '''
//                             }
//                         }
//                     }
//                 }
//             }
//         }

//         stage('Build k8s from ECR image') {
//             steps {
//                 script {
//                     withCredentials([
//                         string(credentialsId: 'ECR_REPO', variable: 'ECR_REPO')
//                     ]) {
//                         withAWS(credentials: 'aws-jenkins-cred', region: 'us-east-1') {
//                             docker.withRegistry("https://${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com", "ecr:us-east-1:aws-jenkins-cred") {
//                                 sh "kubectl apply -f open-web-ui-deployment.yaml"
//                             }
//                         }
//                     }
//                 }
//             }
//         }
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
