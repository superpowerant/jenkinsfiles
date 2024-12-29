pipeline {
  agent any
  stages {
    stage('pulling code') {
      steps {
        git(url: 'https://github.com/superpowerant/jenkinsfiles.git', branch: '"${BRANCH}"', changelog: true, credentialsId: '851f1073-6597-4b6f-8cba-10544554beb4')
      }
    }

  }
}