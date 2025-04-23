pipeline {

  tools {
        maven 'Maven'
  }

  environment {
    dockerimagename = "brahim2023/spring-boot-k8s"
    dockerImage = ""
  }

  agent any

  stages {

    stage('Build App') {
        steps {
            script {
              try {
                sh 'mvn clean package'
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
            withKubeConfig([credentialsId: 'mykubeconfig', serverUrl: 'https://192.168.49.2:8443']) {
              sh 'kubectl apply -f deployment-k8s.yaml'
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
