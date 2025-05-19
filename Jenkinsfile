pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'preethamreddy1999/nodejs-app'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
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
                    sh "docker push preethamreddy1999/nodejs-app:latest"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh']) {
                    sh 'scp docker-compose.yml ec2-user@13.215.45.205:/home/ec2-user/'
                    sh "ssh ec2-user@13.215.45.205 'docker-compose pull && docker-compose down && docker-compose up -d'"
                }
            }
        }
    }

    post {
        success {
            mail to: 'preethamreddykarrem@gmail.com',
                 subject: "✅ Build #${BUILD_NUMBER} Succeeded",
                 body: "CI/CD pipeline completed successfully."
        }
        failure {
            mail to: 'preethamreddykarrem@gmail.com',
                 subject: "❌ Build #${BUILD_NUMBER} Failed",
                 body: "CI/CD pipeline failed. Please check Jenkins logs."
        }
    }
}
