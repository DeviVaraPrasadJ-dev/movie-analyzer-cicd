pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
    - cat
    tty: true
"""
    }
  }

  environment {
    AWS_REGION = "ap-south-1"
    ECR_ACCOUNT = "433193941125"
    IMAGE_NAME = "movie-backend"
    IMAGE_TAG = "kaniko-test"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Kaniko Build & Push') {
      steps {
        container('kaniko') {
          sh '''
          mkdir -p /kaniko/.docker

          cat <<EOF > /kaniko/.docker/config.json
          {
            "credHelpers": {
              "${ECR_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com": "ecr-login"
            }
          }
          EOF

          /kaniko/executor \
            --context=backend \
            --dockerfile=backend/Dockerfile \
            --destination=${ECR_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:${IMAGE_TAG}
          '''
        }
      }
    }
  }
}
