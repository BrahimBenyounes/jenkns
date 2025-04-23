pipeline {
    agent any

    tools {
        maven 'M3'
        jdk 'jdk17'
    }

    environment {
        dockerimagename = "brahim2025/spring-boot-k8s"
        dockerImage = ""
        MAVEN_HOME = tool name: 'M3', type: 'maven'
        JAVA_HOME = tool name: 'jdk17', type: 'jdk'
        PATH = "${MAVEN_HOME}/bin:${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Build App') {
            steps {
                script {
                    try {
                        bat 'mvn clean package'
                    } catch (Exception e) {
                        error "Maven build failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Build image') {
            steps {
                script {
                    try {
                        dockerImage = docker.build(dockerimagename)
                    } catch (Exception e) {
                        error "Docker build failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Pushing Image') {
            environment {
                registryCredential = 'dockerhub-credentials'
            }
            steps {
                script {
                    try {
                        docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                            dockerImage.push("latest")
                        }
                    } catch (Exception e) {
                        error "Docker push failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    try {
                        withKubeConfig([credentialsId: 'mykubeconfig', serverUrl: 'https://localhost:8443']) {
                            bat 'kubectl apply -f deployment-k8s.yaml'
                        }
                    } catch (Exception e) {
                        error "Kubernetes deployment failed: ${e.getMessage()}"
                    }
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
