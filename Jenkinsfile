pipeline {
    agent any

    environment {
        FLASK_IMAGE = 'flask-app'
        NGINX_IMAGE = 'nginx-app'
        IMAGE_TAG   = 'latest'
    }

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
                    trivy fs --format table \
                    --output reports/trivy-fs-report.txt .
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/trivy-fs-report.txt'
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
                        -t ${FLASK_IMAGE}:${IMAGE_TAG} \
                        -f Dockerfile .
                '''

                sh '''
                    docker build \
                        -t ${NGINX_IMAGE}:${IMAGE_TAG} \
                        -f Dockerfile.nginx .
                '''
            }
        }

        stage('Trivy image scan') {
            steps {
                sh '''
                    trivy image \
                    --format table \
                    --output reports/trivy-flask-image-report.txt \
                    ${FLASK_IMAGE}:${IMAGE_TAG}
                '''

                sh '''
                    trivy image \
                    --format table \
                    --output reports/trivy-nginx-image-report.txt \
                    ${NGINX_IMAGE}:${IMAGE_TAG}
                '''
            }

            post {
                always {
                    archiveArtifacts artifacts: 'reports/*.txt'
                }
            }
        }

        stage('Run containers') {
            steps {
                sh '''
                    docker run -d \
                        --name flask-app \
                        --network app-network \
                        ${FLASK_IMAGE}:${IMAGE_TAG}
                '''

                sh '''
                    docker run -d \
                        --name nginx-app \
                        --network app-network \
                        -p 80:80 \
                        ${NGINX_IMAGE}:${IMAGE_TAG}
                '''
            }
        }
    }
}
