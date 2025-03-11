pipeline {
    agent any

    environment {
        dockerimagename = "brahim2023/spring-boot-k8s"
    }

    tools {
        maven 'Maven'
    }

    stages {
        stage('Build App') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Load Minikube Docker Environment (if using Minikube)
                    sh 'eval $(minikube docker-env)'
                    
                    // Build Docker Image and store it in a variable
                    dockerImage = docker.build("${dockerimagename}:latest")
                }
            }
        }

        stage('Push Docker Image') {
            environment {
                registryCredential = 'dockerhub-credentials'  // Jenkins stored credentials
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'mykubeconfig', serverUrl: 'https://192.168.49.2:8443']) {
                    sh 'kubectl apply -f deployment-k8s.yaml'
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Deployment failed. Check logs.'
        }
    }
}
