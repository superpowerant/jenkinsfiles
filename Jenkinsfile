pipeline {
  agent any
  stages {
    stage('Pulling') {
      parallel {
        stage('pulling code') {
          steps {
            git(url: '${REPO_URL}', branch: '${BRANCH}', changelog: true, credentialsId: '851f1073-6597-4b6f-8cba-10544554beb4')
          }
        }

        stage('trigger') {
          steps {
            git(url: 'https://github.com/superpowerant/jenkinsfiles.git', changelog: true, branch: 'env.gitlabBranch', credentialsId: '851f1073-6597-4b6f-8cba-10544554beb4')
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
          TAG = curDate[0..14] + "-" + CommitID + "-" + BRANCH
        }

      }
    }

    stage('Build') {
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

    stage('BuildContainer') {
      steps {
        container(name: 'BuilderContainer') {
          sh '''
echo "building start"
${BUILD_COMMAND}
'''
        }

      }
    }

    stage('Image') {
      steps {
        sh '''
docker build -t ${HARBOR_ADDRESS}/${PROJECT_NAME}/${IMAGE_NAME}:${TAG} .
docker login -u ${HARBOR_NAME} -p ${HARBOR_PWD} ${HARBOR_ADDRESS}
docker push ${HARBOR_ADDRESS}/${PROJECT_NAME}/${IMAGE_NAME}:${TAG}
'''
      }
    }

    stage('ImageContainer') {
      steps {
        container(name: 'ImageContainer') {
          sh '''
docker build -t ${HARBOR_ADDRESS}/${PROJECT_NAME}/${IMAGE_NAME}:${TAG} .
docker login -u ${HARBOR_NAME} -p ${HARBOR_PWD} ${HARBOR_ADDRESS}
docker push ${HARBOR_ADDRESS}/${PROJECT_NAME}/${IMAGE_NAME}:${TAG}
'''
        }

      }
    }

    stage('DeployContainer') {
      steps {
        container(name: 'kubctl') {
          sh '''
kubectl config use-context ${CLUSTER} --kubeconfig=${KUBECONFIG_PATH}
kubectl set image ${DEPLOY_TYPE} -l ${RESOURCE_LABEL} ${IMAGE_NAME}=${HARBOR_ADDRESS}/${PROJECT_NAME}/${IMAGE_NAME}:${TAG} -n ${NAMESPACE}
'''
        }

      }
    }

  }
  environment {
    CommitMessage = ''
    CommitID = ''
    TAG = ''
  }
}