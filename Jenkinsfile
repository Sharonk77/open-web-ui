pipeline {
   agent any

   stages {
      stage('Clone and Build Docker Image') {
         steps {
            sh '''
            echo pwd
            echo "Starting to build image..."
            git clone https://github.com/open-webui/open-webui
            cd open-webui
            docker build -t open-web-ui-image .
            '''
         }
      }
   }
}
