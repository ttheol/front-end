
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
    DOCKER_COMPOSE_TEMPLATE="$WORKSPACE/infrastructure/infrastructure/neoload/docker-compose.template"
    DOCKER_COMPOSE_LG_FILE = "$WORKSPACE/infrastructure/infrastructure/neoload/docker-compose-neoload.yml"
    HOST="ec2-34-245-206-132.eu-west-1.compute.amazonaws.com"
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
                                         sh "cp -f ${DOCKER_COMPOSE_LG_TEMPLATE_FILE} ${DOCKER_COMPOSE_LG_FILE}"
                                         sh "sed -i 's,TO_REPLACE,${APP_NAME},'  ${DOCKER_COMPOSE_LG_FILE}"
                                         sh "sed -i 's,TOKEN_TOBE_REPLACE,$NLAPIKEY,'  ${DOCKER_COMPOSE_LG_FILE}"
                                         sh 'docker-compose -f ${DOCKER_COMPOSE_LG_FILE} up -d'
                                         sleep 15

                                     }

                         }




     stage('Run load test') {

            steps {

                     sh "cp $WORKSPACE/monspec/frontend_anomalieDection.json $WORKSPACE/test/neoload/load_template/custom-resources/"


                     sh "sed -i 's/HOST_TO_REPLACE/${HOST}/'  $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's/PORT_TO_REPLACE/80/' $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's/DTID_TO_REPLACE/${DYNATRACEID}/' $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's/APIKEY_TO_REPLACE/${DYNATRACEAPIKEY}/'  $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's,JSONFILE_TO_REPLACE,front-end_monspec.json,'  $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's/TAGS_TO_REPLACE/${NL_DT_TAG}/' $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's,OUTPUTFILE_TO_REPLACE,$WORKSPACE/infrastructure/sanitycheck.json,'  $WORKSPACE/test/neoload/Frontend_neoload.yaml"


                    sh "mkdir $WORKSPACE/test/neoload/neoload_project"
                    sh "cp $WORKSPACE/test/neoload/Frontend_neoload.yaml $WORKSPACE/test/neoload/load_template/"
                    sh "cd $WORKSPACE/test/neoload/load_template/ ; zip -r $WORKSPACE/test/neoload/neoload_project/neoloadproject.zip ./*"


                    sh "docker run --rm \
                                             -v $WORKSPACE/test/neoload/neoload_project/:/neoload-project \
                                             -e NEOLOADWEB_TOKEN=$NLAPIKEY \
                                             -e TEST_RESULT_NAME=Stage_load_${VERSION}_${BUILD_NUMBER} \
                                             -e SCENARIO_NAME=FrontEndLoad \
                                             -e CONTROLLER_ZONE_ID=defaultzone \
                                             -e LG_ZONE_IDS=defaultzone:1 \
                                             -e AS_CODE_FILES=Frontend_neoload.yaml \
                                             --network ${APP_NAME} \
                                              neotys/neoload-web-test-launcher:latest"
                      /*script {
                          neoloadRun executable: '/home/neoload/neoload/bin/NeoLoadCmd',
                                  project: "$WORKSPACE/test/neoload/load_template/load_template.nlp",
                                  testName: 'Stage_load_${VERSION}_${BUILD_NUMBER}',
                                  testDescription: 'Stage_load_${VERSION}_${BUILD_NUMBER}',
                                  commandLineOption: "-project  $WORKSPACE/test/neoload/Frontend_neoload.yaml -nlweb -L Population_Buyer=$WORKSPACE/infrastructure/infrastructure/neoload/lg/remote.txt -L Population_Dynatrace_Integration=$WORKSPACE/infrastructure/infrastructure/neoload/lg/local.txt -nlwebToken $NLAPIKEY -variables host=ec2-54-229-141-49.eu-west-1.compute.amazonaws.com,port=80",
                                  scenario: 'FrontEndLoad', sharedLicense: [server: 'NeoLoad Demo License', duration: 2, vuCount: 200],
                                  trendGraphs: [
                                          [name: 'Limit test Catalogue API Response time', curve: ['CatalogueList>Actions>Get Catalogue List'], statistic: 'average'],
                                          'ErrorRate'
                                  ]
                      }*/

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

