
pipeline {
  agent {
    label 'master'
  }
  environment {
    VERSION="0.1"
    APP_NAME = "front-end"
    TAG = "neotysdevopsdemo/${APP_NAME}"
    TAG_DEV = "${TAG}:DEV-${VERSION}"
    TAG_STAGING = "${TAG}-stagging:${VERSION}"
    GROUP="neotysdevopsdemo"
    DYNATRACEID="https://${env.DT_ACCOUNTID}.live.dynatrace.com/"
    DYNATRACEAPIKEY="${env.DT_API_TOKEN}"
    NLAPIKEY="${env.NL_WEB_API_KEY}"
    NL_DT_TAG="app:${env.APP_NAME}"
    DOCKER_COMPOSE_TEMPLATE="$WORKSPACE/infrastructure/infrastructure/neoload/docker-compose.template"
    DOCKER_COMPOSE_LG_FILE = "$WORKSPACE/infrastructure/infrastructure/neoload/docker-compose-neoload.yml"
    HOST="ec2-52-50-215-174.eu-west-1.compute.amazonaws.com"
  }
  stages {
   /* stage('Checkout') {
          agent { label 'master' }
          steps {
              git  url:"https://github.com/${GROUP}/${APP_NAME}.git",
                      branch :'master'
          }
      }
    stage('build app')
    {
      agent {
            docker { image 'eliostech/jenkins-slave-nodejs:latest'
              reuseNode true
              }
        }
        steps{
            sh 'npm install'
        }
    }*/
    stage('Docker build') {

      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'TOKEN', usernameVariable: 'USER')]) {
          sh "docker build -t ${GROUP}/${APP_NAME}:DEV-0.1 ."
          sh "docker tag ${GROUP}/${APP_NAME}:DEV-0.1 ${TAG_DEV}"
          sh "docker login --username=${USER} --password=${TOKEN}"
          sh "docker push ${TAG_DEV}"
        }

      }
    }

    stage('create docker network') {

          steps {
               sh "docker network create ${APP_NAME} || true"

          }
   }

    stage('Deploy to dev namespace') {
        steps {
          sh "sed -i 's,VERSION_TO_REPLACE,${VERSION},'  $WORKSPACE/docker-compose.yml"
          sh "sed -i 's,TO_REPLACE,${APP_NAME},' $WORKSPACE/docker-compose.yml"
          sh 'docker-compose -f $WORKSPACE/docker-compose.yml up -d'

        }

    }
   stage('Start NeoLoad infrastructure') {

                              steps {
                                         sh "cp -f ${DOCKER_COMPOSE_TEMPLATE} ${DOCKER_COMPOSE_LG_FILE}"
                                         sh "sed -i 's,TO_REPLACE,${APP_NAME},'  ${DOCKER_COMPOSE_LG_FILE}"
                                         sh "sed -i 's,TOKEN_TOBE_REPLACE,$NLAPIKEY,'  ${DOCKER_COMPOSE_LG_FILE}"
                                         sh 'docker-compose -f ${DOCKER_COMPOSE_LG_FILE} up -d'
                                         sleep 15

                                     }

                         }



      stage('generate traffic')
      {
         steps{
            sh "curl http://${HOST}:80"
            sh "curl http://${HOST}:80"
            sh "curl http://${HOST}:80"
         }
      }
      stage('NeoLoad Test')
     {
      agent {
      docker {
          image 'python:3-alpine'
          reuseNode true
       }

         }
     stages {
          stage('Get NeoLoad CLI') {
                       steps {
                         withEnv(["HOME=${env.WORKSPACE}"]) {

                          sh '''
                               export PATH=~/.local/bin:$PATH
                               pip3 install neoload
                               neoload --version
                           '''

                         }
                       }
         }


         stage('Run load test') {

                steps {
                     withEnv(["HOME=${env.WORKSPACE}"]) {

                         sh "sed -i 's/HOST_TO_REPLACE/${HOST}/'  $WORKSPACE/test/neoload/load_template/Frontend_neoload.yaml"
                         sh "sed -i 's/PORT_TO_REPLACE/80/' $WORKSPACE/test/neoload/load_template/Frontend_neoload.yaml"
                         sh "sed -i 's,DTID_TO_REPLACE,${DYNATRACEID},' $WORKSPACE/test/neoload/load_template/Frontend_neoload.yaml"
                         sh "sed -i 's/APIKEY_TO_REPLACE/${DYNATRACEAPIKEY}/'  $WORKSPACE/test/neoload/load_template/Frontend_neoload.yaml"
                         sh "sed -i 's/TAGS_TO_REPLACE/${NL_DT_TAG}/' $WORKSPACE/test/neoload/load_template/Frontend_neoload.yaml"


                         sh """
                               export PATH=~/.local/bin:$PATH
                               neoload \
                               login --workspace "Default Workspace" $NLAPIKEY \
                               test-settings  --zone defaultzone --scenario FrontEndLoad use FrontDynatrace \
                               project --path $WORKSPACE/test/neoload/load_template/ upload
                          """


                        }

                      }
        }
         stage('Run Test') {
              steps {
                withEnv(["HOME=${env.WORKSPACE}"]) {
                  sh """
                       export PATH=~/.local/bin:$PATH
                       neoload run \
                      --return-0 \
                        --as-code Frontend_neoload.yaml \
                        FrontDynatrace
                     """
                }
              }
         }
         stage('Generate Test Report') {
              steps {
                withEnv(["HOME=${env.WORKSPACE}"]) {
                    sh """
                         export PATH=~/.local/bin:$PATH
                         neoload test-results junitsla
                       """
                }
              }
              post {
                  always {
                      junit 'junit-sla.xml'
                  }
              }
        }
      }
    }

    }
    post {
            always {
                    sh 'docker-compose -f $WORKSPACE/infrastructure/infrastructure/neoload/lg/docker-compose.yml down'
                    cleanWs()
                    sh 'docker volume prune'
            }

          }
  }

