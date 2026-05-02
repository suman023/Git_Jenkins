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
                sh 'sudo docker build -t suman2304/javatest -f dockerfile .'
            }
        }

        stage('login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | sudo docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('push') {
            steps {
                sh 'sudo docker push suman2304/javatest'
            }
        }
    }
}
