
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
    GROUP="neotysdevopsdemowithvirtualize"
  }
  stages {
    stage('Checkout') {
          agent { label 'master' }
          steps {
              git  url:"https://github.com/${GROUP}/${APP_NAME}.git",
                      branch :'master'
          }
      }
    stage('build app')
    {
      agent {
            docker { image 'eliostech/jenkins-slave-nodejs' }
        }
        steps{
            sh 'npm install'
        }
    }
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



  /*  stage('DT Deploy Event') {
        when {
            expression {
            return env.BRANCH_NAME ==~ 'release/.*' || env.BRANCH_NAME ==~'master'
            }
        }
        steps {
          container("curl") {
            // send custom deployment event to Dynatrace
            sh "curl -X POST \"$DT_TENANT_URL/api/v1/events?Api-Token=$DT_API_TOKEN\" -H \"accept: application/json\" -H \"Content-Type: application/json\" -d \"{ \\\"eventType\\\": \\\"CUSTOM_DEPLOYMENT\\\", \\\"attachRules\\\": { \\\"tagRule\\\" : [{ \\\"meTypes\\\" : [\\\"SERVICE\\\"], \\\"tags\\\" : [ { \\\"context\\\" : \\\"CONTEXTLESS\\\", \\\"key\\\" : \\\"app\\\", \\\"value\\\" : \\\"${env.APP_NAME}\\\" }, { \\\"context\\\" : \\\"CONTEXTLESS\\\", \\\"key\\\" : \\\"environment\\\", \\\"value\\\" : \\\"dev\\\" } ] }] }, \\\"deploymentName\\\":\\\"${env.JOB_NAME}\\\", \\\"deploymentVersion\\\":\\\"${_VERSION}\\\", \\\"deploymentProject\\\":\\\"\\\", \\\"ciBackLink\\\":\\\"${env.BUILD_URL}\\\", \\\"source\\\":\\\"Jenkins\\\", \\\"customProperties\\\": { \\\"Jenkins Build Number\\\": \\\"${env.BUILD_ID}\\\",  \\\"Git commit\\\": \\\"${env.GIT_COMMIT}\\\" } }\" "
          }
        }
    }*/
    /*stage('Mark artifact for staging namespace') {

      steps {

          sh "docker tag ${TAG_DEV} ${TAG_STAGING}"
          sh "docker push ${TAG_STAGING}"

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

