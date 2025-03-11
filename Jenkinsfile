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
                    def dockerImage = docker.build("${dockerimagename}:latest")
                }
            }
        }

        stage('Push Docker Image') {
            environment {
                registryCredential = 'dockerhub-credentials'  // Using Jenkins stored credentials
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        def dockerImage = docker.build("${dockerimagename}:latest")
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
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
