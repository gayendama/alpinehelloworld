pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"                    /*alpinehelloworld par exemple*/
        APP_EXPOSED_PORT = "80"            /*80 par défaut*/
        APP_NAME = "eazytraining"                        /*eazytraining par exemple*/
        IMAGE_TAG = "latest"                      /*tag docker, par exemple latest*/
        DOCKERHUB_ID = "ndamagaye"
        DOCKERHUB_PASSWORD = ""
        INTERNAL_PORT = "5000"              /*5000 par défaut*/
        EXTERNAL_PORT = "80"
        CONTAINER_IMAGE = "${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    agent none
    stages {
       stage('Build image') {
           agent any
           steps {
              script {
                sh 'docker build -t ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG .'
              }
           }
       }
       stage('Run container based on builded image') {
          agent any
          steps {
            script {
              sh '''
                  echo "Cleaning existing container if exist"
                  docker ps -a | grep -i $IMAGE_NAME && docker rm -f $IMAGE_NAME
                  docker run --name $IMAGE_NAME -d -p $APP_EXPOSED_PORT:$INTERNAL_PORT  -e PORT=$INTERNAL_PORT ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
                  sleep 5
              '''
             }
          }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                   curl -v 127.0.0.1:$APP_EXPOSED_PORT | grep -q "Hello world!"
                '''
              }
           }
       }
       stage('Clean container') {
          agent any
          steps {
             script {
               sh '''
                   docker stop $IMAGE_NAME
                   docker rm $IMAGE_NAME
               '''
             }
          }
      }

      
      
     
}
