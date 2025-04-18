pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'sparwal175/health_care_app:v0.2'
        APP_SERVER = 'ec2-user@34.233.120.123'
    }
 
    stages {
       stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sparwal175/healthcare_medilife.git'
            }
        }
 
        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker build -t $DOCKER_IMAGE .
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }
 
        stage('Deploy on App Server') {
            steps {
                sshagent(credentials: ['app-server-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no $APP_SERVER "
                            sudo docker pull $DOCKER_IMAGE &&
                            sudo docker stop myapp || true &&
                            sudo docker rm myapp || true &&
                            sudo docker run -d --name myapp -p 80:80 $DOCKER_IMAGE
                        "
                    '''
                }
            }
        }
    }
}