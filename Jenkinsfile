
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


    stage('Deploy to dev namespace') {
        steps {
          sh "sed -i 's,VERSION_TO_REPLACE,${VERSION},'  $WORKSPACE/docker-compose.yml"
          sh 'docker-compose -f $WORKSPACE/docker-compose.yml up -d'

        }

    }
    stage('Start NeoLoad infrastructure') {

              steps {
                  sh 'docker-compose -f $WORKSPACE/infrastructure/infrastructure/neoload/lg/docker-compose.yml up -d'

              }

          }




     stage('Run load test') {
            agent {
                dockerfile {
                    args '--user root -v /tmp:/tmp --network=parasoft'
                    dir 'infrastructure/infrastructure/neoload/controller/'
                }
            }
            steps {

                     sh "sed -i 's/HOST_TO_REPLACE/${HOST}/'  $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's/PORT_TO_REPLACE/80/' $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's/DTID_TO_REPLACE/${DYNATRACEID}/' $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's/APIKEY_TO_REPLACE/${DYNATRACEAPIKEY}/'  $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's,JSONFILE_TO_REPLACE,$WORKSPACE/monspec/front-end_monspec.json,'  $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's/TAGS_TO_REPLACE/${NL_DT_TAG}/' $WORKSPACE/test/neoload/Frontend_neoload.yaml"
                     sh "sed -i 's,OUTPUTFILE_TO_REPLACE,$WORKSPACE/infrastructure/sanitycheck.json,'  $WORKSPACE/test/neoload/Frontend_neoload.yaml"

                      script {
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
                      }

                  }
                }*/


    }
    post {
            always {
                    sh 'docker-compose -f $WORKSPACE/infrastructure/infrastructure/neoload/lg/docker-compose.yml down'
                    cleanWs()
                    sh 'docker volume prune'
            }

          }
  }

