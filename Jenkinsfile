pipeline {
  agent {
  kubernetes {
    yaml """
apiVersion: v1
kind: Pod
metadata:
  annotations:
    jenkins.io/skip-container-readiness: "kaniko"
spec:
  serviceAccountName: jenkins
  restartPolicy: Never
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - /busybox/sh
    args:
    - -c
    - sleep 3600
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
      sh '''
        /kaniko/executor \
        --context=dir://${WORKSPACE}/movie-analyzer-app/backend \
        --dockerfile=${WORKSPACE}/movie-analyzer-app/backend/Dockerfile \
        --destination=${ECR_REGISTRY}/movie-backend:${BUILD_NUMBER} \
        --verbosity=info

      '''
    }
  }
}

    stage('Build & Push Frontend') {
      steps {
        container('kaniko') {
          sh """
          /kaniko/executor \
            --context=dir://${WORKSPACE}/movie-analyzer-app/frontend \
            --dockerfile=${WORKSPACE}/movie-analyzer-app/frontend/Dockerfile \
            --destination=${ECR_REGISTRY}/movie-frontend:${BUILD_NUMBER}

          """
        }
      }
    }

    stage('Build & Push Model') {
      steps {
        container('kaniko') {
          sh """
           /kaniko/executor \
            --context=dir://${WORKSPACE}/movie-analyzer-app/model \
            --dockerfile=${WORKSPACE}/movie-analyzer-app/model/Dockerfile \
            --destination=${ECR_REGISTRY}/movie-model:${BUILD_NUMBER}

          """
        }
      }
    }
  }
}
