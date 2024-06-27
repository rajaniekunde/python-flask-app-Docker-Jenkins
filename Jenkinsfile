pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'your-app:latest'
        GIT_URL = 'https://github.com/rajaniekunde/python-flask-app-Docker-Jenkins.git'
        GIT_BRANCH = 'main'
        CONTAINER_NAME = 'your-app'
        EC2_HOST = '54.242.196.0' // Replace with your EC2 instance's public DNS/IP
        EC2_USER = 'ubuntu' // Replace with your EC2 instance's SSH username
        SSH_CREDENTIALS = 'admin' // Jenkins credentials ID for SSH key
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: GIT_BRANCH, credentialsId: 'admin', url: GIT_URL
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build DOCKER_IMAGE
                }
            }
        }
        
        stage('Run Container Locally') {
            steps {
                script {
                    docker.run("-d --name ${CONTAINER_NAME} -p 5000:5000 ${DOCKER_IMAGE}")
                }
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                script {
                    sshagent(credentials: ['${SSH_CREDENTIALS}']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'docker stop ${CONTAINER_NAME} || true'
                            ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'docker rm ${CONTAINER_NAME} || true'
                            ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'docker pull ${DOCKER_IMAGE}'
                            ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'docker run -d --name ${CONTAINER_NAME} -p 80:5000 ${DOCKER_IMAGE}'  // Adjusted port from 5000:5000 to 80:5000
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
