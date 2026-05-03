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
				sh ' trivy image --format json --output result.json  --severity HIGH,CRITICAL  suman2304/javatest'
				archiveArtifacts artifacts: 'result.json', followSymlinks: false
              //  sh 'trivy image --exit-code 0 --severity LOW,MEDIUM suman2304/javatest'
               // sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL suman2304/javatest'
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
stage('mailing') {
            steps {
	mail bcc: '', body: 'echo jobstatus ', cc: '', from: 'sumanshit023@gmail.com', replyTo: '', subject: 'Pipeline status', to: 'sumanshit023@gmail.com'
        }
 }
    }

    post {
        always {
            sh 'sudo docker logout'
        }
    }
}
