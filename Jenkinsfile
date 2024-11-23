pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'trongpham99/server_golang'
        DOCKER_TAG = 'latest'
        PROD_SERVER = 'ec2-47-129-57-10.ap-southeast-1.compute.amazonaws.com'
        PROD_USER = 'ubuntu'
        TELEGRAM_BOT_TOKEN = '7609228495:AAFwXuFZxMIen4sydKWJirXf3UKuu7uzHJ4'
        TELEGRAM_CHAT_ID = '-1002441532713'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/trongpham99-cpu/server_golang_v2.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image for linux/amd64 platform...'
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", "--platform linux/amd64 .")
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credential') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy Golang to DEV') {
            steps {
                script {
                    echo 'Clearing server_golang-related images and containers...'
                    sh '''
                        docker container stop server-golang || echo "No container named server-golang to stop"
                        docker container rm server-golang || echo "No container named server-golang to remove"
                        docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG} || echo "No image ${DOCKER_IMAGE}:${DOCKER_TAG} to remove"
                    '''
                    
                    echo 'Deploying to DEV environment...'
                    sh '''
                        docker image pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker network create dev || echo "Network already exists"
                        docker container run -d --rm --name server-golang -p 3000:3000 --network dev ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Production on AWS') {
            steps {
                script {
                    echo 'Deploying to Production...'
                    sshagent(['aws-ssh-key']) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no ${PROD_USER}@${PROD_SERVER} << EOF
                                docker container stop server-golang || echo "No container to stop"
                                docker container rm server-golang || echo "No container to remove"
                                docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG} || echo "No image to remove"
                                docker image pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                                docker container run -d --rm --name server-golang -p 3000:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }

        success {
            sendTelegramMessage("✅ Build #${BUILD_NUMBER} was successful! ✅")
        }

        failure {
            sendTelegramMessage("❌ Build #${BUILD_NUMBER} failed. ❌")
        }
    }
}

def sendTelegramMessage(String message) {
    sh """
    curl -s -X POST https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage \
    -d chat_id=${TELEGRAM_CHAT_ID} \
    -d text="${message}"
    """
}