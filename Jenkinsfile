pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'preethamreddy1999/nodejs-app'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'  // Docker Hub credentials ID
        SSH_CREDENTIALS_ID = 'ec2-ssh'             // SSH key credentials ID for EC2
        EC2_USER = 'ec2-user'
        EC2_HOST = '13.215.45.205'
        DEPLOY_PATH = '/home/ec2-user'
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy via SCP') {
            steps {
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    // Replace 'your-app' and 'user@host:/path/' with actual file and destination
                    sh "scp your-app ${EC2_USER}@${EC2_HOST}:${DEPLOY_PATH}/"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh "scp docker-compose.yml ${EC2_USER}@${EC2_HOST}:${DEPLOY_PATH}/"
                    sh "ssh ${EC2_USER}@${EC2_HOST} 'cd ${DEPLOY_PATH} && docker-compose pull && docker-compose down && docker-compose up -d'"
                }
            }
        }
    }
}
