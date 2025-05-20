pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'preethamreddy1999/nodejs-app'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'  // Docker Hub creds ID
        SSH_CREDENTIALS_ID = 'ec2-ssh'              // SSH key credentials ID
        DEPLOY_USER = 'ubuntu'
        DEPLOY_HOST = '13.215.45.205'
        DEPLOY_PATH = '/home/ubuntu'
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

        stage('Copy Files to EC2') {
            steps {
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    // Only copy what's needed
                    sh """
                        scp -o StrictHostKeyChecking=no Dockerfile Jenkinsfile docker-compose.yml index.js package.json package-lock.json ${env.DEPLOY_USER}@${env.DEPLOY_HOST}:${env.DEPLOY_PATH}
                    """
                }
            }
        }

        stage('Install Dependencies on EC2') {
            steps {
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.DEPLOY_USER}@${env.DEPLOY_HOST} 'cd ${env.DEPLOY_PATH} && npm install'
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.DEPLOY_USER}@${env.DEPLOY_HOST} '
                            cd ${env.DEPLOY_PATH} &&
                            docker-compose pull &&
                            docker-compose down &&
                            docker-compose up -d
                        '
                    """
                }
            }
        }
    }
}
