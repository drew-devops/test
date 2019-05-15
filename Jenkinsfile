pipeline {
  agent {
    docker {
      label 'jenkins-worker'
      image 'jenkins:jnlp-slave'
    }
  }
  stages {
    stage('Build') {
      steps {
        sh 'sleep 300'
      }
    }
  }
}
