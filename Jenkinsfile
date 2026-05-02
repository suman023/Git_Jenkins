pipeline {
    agent any

    stages {
        stage('clone') {
            steps {
               git 'https://github.com/openwriteup/java11-jenkins.git'
            }
        }
	  stage('build') {
            steps {
               sh '''sudo docker build -t suman2304/javatest -f dockerfile .
'''
            }
        }

    }
}
