pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"                    /*alpinehelloworld par exemple*/
        APP_EXPOSED_PORT = "80"            /*80 par défaut*/
        APP_NAME = "eazytraining"                        /*eazytraining par exemple*/
        IMAGE_TAG = "latest"                      /*tag docker, par exemple latest*/
        STAGING = "env-staging"
        PRODUCTION = "env-prod"
        DOCKERHUB_ID = "ndamagaye"
        DOCKERHUB_PASSWORD = "Sokhn@diaw"
        STG_API_ENDPOINT = "ip10-0-0-3-cjlj0d979sugqdpn18dg-1993.direct.docker.labs.eazytraining.fr"        /* Mettre le couple IP:PORT de votre API eazylabs, exemple 100.25.147.76:1993 */
        STG_APP_ENDPOINT = "ip10-0-0-3-cjlj0d979sugqdpn18dg-80.direct.docker.labs.eazytraining.fr"        /* Mettre le couple IP:PORT votre application en staging, exemple 100.25.147.76:8000 */
        PROD_API_ENDPOINT = "ip10-0-4-4-cjlj0d979sugqdpn18dg-1993.direct.docker.labs.eazytraining.fr"      /* Mettre le couple IP:PORT de votre API eazylabs, 100.25.147.76:1993 */
        PROD_APP_ENDPOINT = "ip10-0-0-4-cjlj0d979sugqdpn18dg-80.direct.docker.labs.eazytraining.fr"      /* Mettre le couple IP:PORT votre application en production, exemple 100.25.147.76 */
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

      stage ('Login and Push Image on docker hub') {
          agent any
          steps {
             script {
               sh '''
                   echo $DOCKERHUB_PASSWORD_PSW | sudo docker login -u $DOCKERHUB_ID_USR 
                   docker push ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
      }

      stage('STAGING - Deploy app') {
      agent any
      steps {
          script {
            sh """
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}00\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
              curl -v -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
            """
          }
        }
     
     }
     stage('PROD - Deploy app') {
       when {
           expression { GIT_BRANCH == 'origin/main' }
       }
     agent any

       steps {
          script {
            sh """
              echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
              curl -v -X POST http://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
            """
          }
       }
     }
  }
}
