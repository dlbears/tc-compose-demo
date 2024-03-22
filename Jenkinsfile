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
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@${EC2_INSTANCE} 'docker-compose down'"
                    sh "rsync -avz --delete . ec2-user@${EC2_INSTANCE}:/path/to/destination"
                    sh "ssh ec2-user@${EC2_INSTANCE} 'docker pull ${DOCKERHUB_REPOSITORY}:${env.BUILD_NUMBER}'"
                    sh "ssh ec2-user@${EC2_INSTANCE} 'docker-compose up -d'"
                }
            }
        }
    }
}
