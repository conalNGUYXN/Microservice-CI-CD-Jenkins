pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('a0d02fc9-ed65-4978-abff-2a9b01cc56ae')
        DOCKERHUB_USER = "${DOCKERHUB_CREDENTIALS_USR}"
        DOCKERHUB_PASS = "${DOCKERHUB_CREDENTIALS_PSW}"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/conalNGUYXN/Microservice-CI-CD-Jenkins.git']]
                ])
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    script {
                        sh "docker build -t ${DOCKERHUB_USER}/frontend:${IMAGE_TAG} ."
                    }
                }
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    script {
                        sh "docker build -t ${DOCKERHUB_USER}/backend:${IMAGE_TAG} ."
                    }
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            steps {
                script {
                    sh "echo ${DOCKERHUB_PASS} | docker login -u ${DOCKERHUB_USER} --password-stdin"
                    sh "docker push ${DOCKERHUB_USER}/frontend:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USER}/backend:${IMAGE_TAG}"
                }
            }
        }

        stage('Test Backend') {
            steps {
                dir('backend') {
                    script {
                        echo 'Running backend tests with unittest...'
                        sh 'python3 -m unittest test_backend.py'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes with Kustomize') {
            steps {
                dir('k8s') {
                    script {
                        echo 'Deploying microservices to Kubernetes using Kustomize...'
                        sh 'kubectl apply -k .'
                    }
                }
            }
        }

        stage('Post Cleanup') {
            steps {
                script {
                    echo "Cleaning up Docker artifacts"
                    sh 'docker system prune -f'
                }
            }
        }
    }
}