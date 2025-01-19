pipeline {
   agent any

   environment {
      //todo: Add any necessary environment variables here, if needed
   }

   stages {
      stage('Clone and Build Docker Image') {
         steps {
            sh '''
            echo "Starting to build image..."
            docker build -t open-web-ui-image https://github.com/open-webui/open-webui
            '''
         }
      }
   }
}
