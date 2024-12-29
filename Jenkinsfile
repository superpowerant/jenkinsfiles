pipeline {
  agent any
  // agent {
  //   kubernetes {
  //     cloud '${CLOUD_NAME}'
  //     slaveConnectTimeout 1200
  //     yaml '''
  //     '''
  //   }
  // }
  stages {
    stage('Pulling') {
      parallel {
        stage('pulling code') {
          when {
            expression {
              env.gitlabBranch == null
            }
          }
          steps {
            git(url: '${REPO_URL}', branch: '${BRANCH}', changelog: true, credentialsId: '41042c6b-2c94-4dbd-9405-f4f1ecaa7f2d')
          }
        }

        stage('trigger') {
          when {
            expression {
              env.gitlabBranch != null
            }
          }
          steps {
            git(url: '${REPO_URL}', changelog: true, branch: env.gitlabBranch, credentialsId: '41042c6b-2c94-4dbd-9405-f4f1ecaa7f2d')
            script {
              BRANCH = env.gitlabBranch
            }

          }
        }

      }
    }

    stage('InitConfig') {
      steps {
        script {
          println "init configration"
          CommitMessage = sh(returnStdOut: true, script: "git log -I --pretty=format:'%h : %an %s'").trim()
          CommitID = sh(returnStdOut: true, script: "git log -I --pretty=format:'%h'").trim()
          def curDate = sh(returnStdOut: true, script: "date '+%Y%m%d-%H%M%S'").trim()
          TAG = ENV_NAME + "-" + curDate[0..14] + "-" + CommitID + "-" + BRANCH
        }

      }
    }

    stage('BuildInLocal') {
      parallel {
        stage('Build') {
          steps {
            sh '''
echo "building start"
${BUILD_COMMAND}
'''
          }
        }

        stage('Scan') {
          steps {
            sh '''
echo "ode scanning"
${SCANNING_COMMAND}'''
          }
        }

      }
    }

//     stage('BuildInContainer') {
//       steps {
//         container(name: 'BuilderContainer') {
//           sh '''
// echo "building start"
// ${BUILD_COMMAND}
// '''
//         }

//       }
//     }

    stage('ImageBuildInLocal') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'ded846e3-842f-4aee-982c-a3100a9c924f', passwordVariable: 'Password', usernameVariable: 'Username')]) {
          sh '''
docker build -t ${HARBOR_ADDRESS}/${PROJECT_NAME}/${IMAGE_NAME}:${TAG} .
docker login -u ${Username} -p ${Password} ${HARBOR_ADDRESS}
docker push ${HARBOR_ADDRESS}/${PROJECT_NAME}/${IMAGE_NAME}:${TAG}
'''
        }
      }
    }

//     stage('ImageBuildInContainer') {
//       steps {
//         withCredentials([usernamePassword(credentialsId: 'ded846e3-842f-4aee-982c-a3100a9c924f', passwordVariable: 'Password', usernameVariable: 'Username')]) {
//           container(name: 'ImageContainer') {
//             sh '''
// docker build -t ${HARBOR_ADDRESS}/${PROJECT_NAME}/${IMAGE_NAME}:${TAG} .
// docker login -u ${Username} -p ${Password} ${HARBOR_ADDRESS}
// docker push ${HARBOR_ADDRESS}/${PROJECT_NAME}/${IMAGE_NAME}:${TAG}
// '''
//           }
//         }
//       }
//     }

//     stage('DeployInContainer') {
//       steps {
//         container(name: 'kubctl') {
//           sh '''
// kubectl config use-context ${CLUSTER} --kubeconfig=${KUBECONFIG_PATH}
// kubectl set image ${DEPLOY_TYPE} -l ${RESOURCE_LABEL} ${IMAGE_NAME}=${HARBOR_ADDRESS}/${PROJECT_NAME}/${IMAGE_NAME}:${TAG} -n ${NAMESPACE}
// '''
//         }

//       }
//     }

  }
  environment {
    CommitMessage = ''
    CommitID = ''
    TAG = ''
  }
}
