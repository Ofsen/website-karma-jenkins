/* import shared library */
@Library('shared-library')_

pipeline {
     environment {
       ID_DOCKER = "${ID_DOCKER_PARAMS}"
       IMAGE_NAME = "website-karma"
       IMAGE_TAG = "latest"
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    echo "Clean Environment"
                    docker rm -f $IMAGE_NAME || echo "container does not exist"
                    docker run --name $IMAGE_NAME -d -p ${PORT_EXPOSED}:80 -e PORT=80 ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
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
                    curl http://127.0.0.1:${PORT_EXPOSED} | grep -q "Deals of the Week"
                '''
              }
           }
      }
      stage('Clean Container') {
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
        environment {
           DOCKERHUB_PASSWORD = credentials('dockerhub')
        }            
          steps {
             script {
               sh '''
                    echo $DOCKERHUB_PASSWORD_PSW | docker login -u ${ID_DOCKER} --password-stdin
                    docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
      }    
     
     stage('Push image in staging and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/main' }
            }
      agent any
      environment {
          RENDER_STAGING_DEPLOY_HOOK = credentials('website-karma-staging')
      }  
      steps {
          script {
            sh '''
               echo "Deploying on staging environment"
               echo $RENDER_STAGING_DEPLOY_HOOK
               curl $RENDER_STAGING_DEPLOY_HOOK
            '''
          }
        }
     }


     stage('Push image in production and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/production' }
            }
      agent any
      environment {
          RENDER_PRODUCTION_DEPLOY_HOOK = credentials('website-karma-production')
      }  
      steps {
          script {
            sh '''
                 echo "Deploying on production environment"
                 curl $RENDER_PRODUCTION_DEPLOY_HOOK
            '''
          }
        }
     }
  }
  post {
    always {
         script {
                emailNotifier currentBuild.result
            }
    }  
  }
}
