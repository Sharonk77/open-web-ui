pipeline {
   agent any

   stages {
      stage('Clone and Build Docker Image') {
         steps {
            sh '''
            pwd
            echo "Starting to clone repository..."
            git clone https://github.com/open-webui/open-webui
            cd open-webui
            echo "Starting to build image..."
            docker build -t open-web-ui-image .
            docker images 
      
            
            '''
         }
      }
   }
}
