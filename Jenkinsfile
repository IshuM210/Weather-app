pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'   // Your DockerHub credential ID in Jenkins
        IMAGE_NAME = "ishwaryamallesh/weather-app"  // Your DockerHub repo
        APP_SERVER = "ubuntu@3.110.184.129"          // Your APP EC2 public IP
        SSH_KEY = "app-ssh-key"                     // Your SSH key credential ID in Jenkins
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master',
                url: 'https://github.com/IshuM210/Weather-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(["${SSH_KEY}"]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${APP_SERVER} '
                        docker pull ${IMAGE_NAME}:latest
                        docker stop weather-app || true
                        docker rm weather-app || true
                        docker run -d -p 5000:5000 --name weather-app ${IMAGE_NAME}:latest
                    '
                    """
                }
            }
        }
    }
}
