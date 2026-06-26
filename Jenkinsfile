pipeline {
    agent any

    stages {
        stage('Clean up') {
            steps {
                sh '''
                    docker rm -f flask-app nginx-app || true
                    docker network rm app-network || true
                '''
            }
        }

        stage('Set up network') {
            steps {
                sh 'docker network create app-network'
            }
        }

        stage('Build images') {
            steps {
                sh 'docker build -t flask-app:latest -f Dockerfile.flask .'
                sh 'docker build -t nginx-app:latest -f Dockerfile.nginx .'
            }
        }

        stage('Run containers') {
            steps {
                sh '''
                    docker run -d --name flask-app --network app-network flask-app:latest

                    docker run -d --name nginx-app \
                      --network app-network \
                      -p 80:80 \
                      nginx-app:latest
                '''
            }
        }
    }
}
