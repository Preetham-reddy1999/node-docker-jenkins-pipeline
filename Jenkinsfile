pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'preethamreddy1999/nodejs-app'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'  // Docker Hub creds ID
        SSH_CREDENTIALS_ID = 'ec2-ssh'              // Your SSH key credentials ID
        DEPLOY_USER = 'ec2-user'
        DEPLOY_HOST = '13.215.45.205'
        DEPLOY_PATH = '/home/ec2-user/'
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
                sh "docker build -t $DOCKER_IMAGE:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push $DOCKER_IMAGE:latest"
                }
            }
        }

        stage('Deploy via SCP') {
            steps {
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    // Replace 'your-app' with actual artifact or folder to deploy
                    sh "scp -o StrictHostKeyChecking=no . ${env.DEPLOY_USER}@${env.DEPLOY_HOST}:${env.DEPLOY_PATH}"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    // Copy docker-compose.yml
                    sh "scp -o StrictHostKeyChecking=no docker-compose.yml ${env.DEPLOY_USER}@${env.DEPLOY_HOST}:${env.DEPLOY_PATH}"
                    // Run docker-compose commands remotely
                    sh "ssh -o StrictHostKeyChecking=no ${env.DEPLOY_USER}@${env.DEPLOY_HOST} 'docker-compose pull && docker-compose down && docker-compose up -d'"
                }
            }
        }
    }
}
