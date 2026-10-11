```groovy
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'cithit/tewelljd'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/tewelljd-cloud/225-lab3-4.git'
        KUBECONFIG = credentials('tewelljd-fl26')
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: "${GITHUB_URL}"]]
                ])
            }
        }

        stage('Lint HTML') {
            steps {
                sh 'npm install htmlhint --save-dev'
                sh 'npx htmlhint *.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry(
                        'https://index.docker.io/v1/',
                        "${DOCKER_CREDENTIALS_ID}"
                    ) {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                script {
                    def kubeConfig = readFile(KUBECONFIG)

                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml"
                    sh 'kubectl apply -f deployment-dev.yaml'
                }
            }
        }

        stage('Deploy to Prod Environment') {
            steps {
                script {
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-prod.yaml"
                    sh 'kubectl apply -f deployment-prod.yaml'
                }
            }
        }

        stage('Check Kubernetes Cluster') {
            steps {
                script {
                    sh 'kubectl get pods'
                    sh 'kubectl get services'
                    sh 'kubectl get deployments'
                    sh 'kubectl get ingress'
                }
            }
        }
    }

    post {
        success {
            slackSend(
                color: 'good',
                message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            )
        }

        unstable {
            slackSend(
                color: 'warning',
                message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            )
        }

        failure {
            slackSend(
                color: 'danger',
                message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            )
        }
    }
}
```