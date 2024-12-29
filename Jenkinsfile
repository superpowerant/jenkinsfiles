pipeline {
  agent any
  stages {
    stage('pulling') {
      parallel {
        stage('pulling code') {
          steps {
            git(url: 'https://github.com/superpowerant/jenkinsfiles.git', branch: '"${BRANCH}"', changelog: true, credentialsId: '851f1073-6597-4b6f-8cba-10544554beb4')
          }
        }

        stage('pulling code by trigger') {
          steps {
            git(url: 'https://github.com/superpowerant/jenkinsfiles.git', changelog: true, branch: 'env.gitlabBranch', credentialsId: '851f1073-6597-4b6f-8cba-10544554beb4')
          }
        }

      }
    }

  }
}