pipeline {
    agent {
        label 'image-worker'
    }
    stages {
        stage('Build') {
            steps {
                echo 'Starting Build'
                sh 'mkdir -p build_output'
                sh 'zip -q -r build_output/build . -x Jenkinsfile *.sh .git* README.md build_output *.zip'
            }
        }
        stage('Publish') {
            steps {
                echo 'Starting Publish'
                sh 'ls -lth build_output'
                sh """
                       cd build_output
                       unzip build.zip
                       ls -lth .
                   """
            }
        }
    }
}
