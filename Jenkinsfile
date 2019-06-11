pipeline {
    agent {
        label 'nodejs-worker'
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
                sh """
			<<-EOF cat | bash
			echo hello
			echo "hello again"
			EOF
                   """
            }
        }
    }
}
