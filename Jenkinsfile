pipeline {
    agent any

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
                        string(credentialsId: 'aws-region', variable: 'AWS_REGION'),
                        string(credentialsId: 'ecr-repo', variable: 'ECR_REPO')
                    ]) {
                        sh '''
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
                pwd
                echo "Starting to clone repository..."
                git clone https://github.com/open-webui/open-webui
                cd open-webui
                echo "Logging into AWS ECR..."
                echo "Starting to build image..."
                docker build -t open-web-ui/openweb .
                '''
            }
        }
    }
}
