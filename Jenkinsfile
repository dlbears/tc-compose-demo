pipeline {
    agent any

    environment {
        DOCKERHUB_REPOSITORY = 'dlbears/tc-wordpress-example'
        EC2_INSTANCE = '10.24.2.140'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        def customImage = docker.build("${DOCKERHUB_REPOSITORY}", "-f Dockerfile.wordpress .")
                        customImage.push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh-credentials']) {
                    sh 'rsync -avz --delete . ec2-user@${EC2_INSTANCE}:/home/ec2-user/wordpress'
                }
                sshagent(credentials: ['ec2-ssh-credentials']) {
                    sh "ssh ec2-user@${EC2_INSTANCE} 'cd /home/ec2-user/wordpress && docker pull ${DOCKERHUB_REPOSITORY}:${env.BUILD_NUMBER} && docker compose down && docker compose up -d'"
                }
            }
        }
    }
}
