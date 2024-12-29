pipeline {
  agent any
  stages {
    stage('Pulling') {
      parallel {
        stage('pulling code') {
          steps {
            git(url: 'https://github.com/superpowerant/jenkinsfiles.git', branch: '"${BRANCH}"', changelog: true, credentialsId: '851f1073-6597-4b6f-8cba-10544554beb4')
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

        stage('Test') {
          steps {
            sh '''
echo "ode scanning"
${SCANNING_COMMAND}'''
          }
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