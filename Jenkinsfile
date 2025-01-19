pipeline {
   agent any

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
