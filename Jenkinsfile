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
    - /busybox/cat
    tty: true
"""
    }
  }

  environment {
    AWS_REGION = "ap-south-1"
    ECR_REGISTRY = "433193941125.dkr.ecr.ap-south-1.amazonaws.com"

    BACKEND_IMAGE  = "\${ECR_REGISTRY}/movie-backend"
    FRONTEND_IMAGE = "\${ECR_REGISTRY}/movie-frontend"
    MODEL_IMAGE    = "\${ECR_REGISTRY}/movie-model"

    IMAGE_TAG = "\${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Push Backend') {
      steps {
        container('kaniko') {
          sh """
          /kaniko/executor \
            --context=movie-analyzer-app/backend \
            --dockerfile=movie-analyzer-app/backend/Dockerfile \
            --destination=\${BACKEND_IMAGE}:\${IMAGE_TAG}
          """
        }
      }
    }

    stage('Build & Push Frontend') {
      steps {
        container('kaniko') {
          sh """
          /kaniko/executor \
            --context=movie-analyzer-app/frontend \
            --dockerfile=movie-analyzer-app/frontend/Dockerfile \
            --destination=\${FRONTEND_IMAGE}:\${IMAGE_TAG}
          """
        }
      }
    }

    stage('Build & Push Model') {
      steps {
        container('kaniko') {
          sh """
          /kaniko/executor \
            --context=movie-analyzer-app/model \
            --dockerfile=movie-analyzer-app/model/Dockerfile \
            --destination=\${MODEL_IMAGE}:\${IMAGE_TAG}
          """
        }
      }
    }
  }
}
