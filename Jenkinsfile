pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "samundra77/nodejs-app"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials" // ID of your Docker Hub credentials in Jenkins
        KUBE_CREDENTIALS_ID = "kubeconfig-id" // ID of your Kubernetes credentials in Jenkins
        IMAGE_TAG = "${env.DOCKER_IMAGE}:${env.BUILD_ID}" // Use build ID or another unique identifier
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("${env.DOCKER_IMAGE}:${env.BUILD_ID}")
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    withDockerRegistry([credentialsId: "${env.DOCKER_CREDENTIALS_ID}", url: 'https://index.docker.io/v1/']) {
                        sh "docker tag ${env.DOCKER_IMAGE}:${env.BUILD_ID} ${env.DOCKER_IMAGE}:latest"
                        sh "docker push ${env.DOCKER_IMAGE}:${env.BUILD_ID}"
                        sh "docker push ${env.DOCKER_IMAGE}:latest"
                    }
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    dockerImage.inside {
                        // Set execute permissions for node_modules/.bin
                        sh 'chmod +x node_modules/.bin/mocha'
                        sh 'npm test'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Create a temporary file with the updated image tag
                    sh "sed 's|IMAGE_TAG|${env.DOCKER_IMAGE}:${env.BUILD_ID}|g' deployment.yaml > k8s-deployment-updated.yaml"
                    kubeconfig(credentialsId: "${env.KUBE_CREDENTIALS_ID}", serverUrl: 'https://0.0.0.0:38429') {
                        sh 'kubectl apply -f deployment.yaml'
                    }
                }
            }
        }
    }
}

