pipeline {
  agent any

  environment {
    registry   = "docker.io"
    reponame   = "kingdodo20"     // Docker Hub username (your repo namespace)
    appname    = "myapp"
    imageTag   = "v${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Image') {
      steps {
        sh """
          docker build -t ${registry}/${reponame}/${appname}:${imageTag} .
          docker images
        """
      }
    }

    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-cred',
                                          usernameVariable: 'DOCKER_USER',
                                          passwordVariable: 'DOCKER_PASS')]) {
          sh """
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin ${registry}
            docker push ${registry}/${reponame}/${appname}:${imageTag}
            docker logout ${registry}
          """
        }
      }
    }

    stage('Deploy to EC2') {
      steps {
        sshagent (credentials: ['ec2-key']) {
          withCredentials([usernamePassword(credentialsId: 'dockerhub-cred',
                                            usernameVariable: 'DOCKER_USER',
                                            passwordVariable: 'DOCKER_PASS')]) {
            sh """
              ssh -o StrictHostKeyChecking=no ubuntu@51.20.108.152 '
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin ${registry}
                docker rm -f myapp || true
                docker pull ${registry}/${reponame}/${appname}:${imageTag}
                docker run -d --name myapp -p 80:80 ${registry}/${reponame}/${appname}:${imageTag}
                docker ps
              '
            """
          }
        }
      }
    }
  }
}
