pipeline {
    agent any

    stages {
        stage('Clean up') {
            steps {
                sh '''
                    docker rm -f flask-app nginx-app || true
                    docker network rm app-network || true
                    mkdir -p reports
                '''
            }
        }

        stage('Trivy filesystem scan') {
            steps {
                sh '''
                    trivy fs --format table --output reports/trivy-fs-report.txt .
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/trivy-fs-report.txt', allowEmptyArchive: true
                }
            }
        }

        stage('Set up network') {
            steps {
                sh 'docker network create app-network'
            }
        }

        stage('Build images') {
            steps {
                sh '''
                    DOCKER_BUILDKIT=1 docker build \
                        -t flask-app:latest \
                        -f Dockerfile .
                '''
        
                sh 'docker build -t nginx-app:latest -f Dockerfile.nginx .'
            }
        }

        stage('Trivy image scan') {
            steps {
                sh '''
                    trivy image --format table --output reports/trivy-flask-image-report.txt flask-app:latest
                    trivy image --format table --output reports/trivy-nginx-image-report.txt nginx-app:latest
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/trivy-*-image-report.txt', allowEmptyArchive: true
                }
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
