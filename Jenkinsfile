pipeline {
    agent any

    stages {

        stage('Clone') {
            steps {
                git 'https://github.com/openwriteup/java11-jenkins.git'
            }
        }

        stage('Build') {
            steps {
                sh 'sudo docker build -t suman2304/javatest -f dockerfile .'
            }
        }

        stage('Install Trivy') {
            steps {
                sh '''
                    sudo apt-get install -y wget apt-transport-https gnupg lsb-release

                    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key \
                        | gpg --dearmor \
                        | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null

                    echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" \
                        | sudo tee /etc/apt/sources.list.d/trivy.list

                    sudo apt-get update
                    sudo apt-get install -y trivy
                    trivy --version
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image --format json --output result.json --severity HIGH,CRITICAL suman2304/javatest'
                archiveArtifacts artifacts: 'result.json', followSymlinks: false
            }
        }

        stage('Login') {
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

        stage('Push') {
            steps {
                sh 'sudo docker push suman2304/javatest'
            }
        }

        stage('Notify') {
            steps {
                // ✅ Email
                mail(
                    to: 'sumanshit023@gmail.com',
                    from: 'sumanshit023@gmail.com',
                    subject: "Pipeline ${currentBuild.result}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "Build Status: ${currentBuild.result}\nJob: ${env.JOB_NAME}\nBuild: ${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}"
                )

                // ✅ Slack 
                slackSend(
            channel: '#jenkinsslackwp',
            color: 'good',
            message: "✅ Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${env.BUILD_URL}"
        )
            }
        }
    }

    post {
        always {
            sh 'sudo docker logout'
        }
        failure {
            slackSend(
                channel: '#jenkinsslackwp',
                color: 'danger',
                message: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${env.BUILD_URL}",
                tokenCredentialId: '6d881e6d-2c2f-4970-be0d-14c526b694d6'
            )
        }
    }
}
