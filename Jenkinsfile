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
    image: gcr.io/kaniko-project/executor:v1.23.2
    imagePullPolicy: IfNotPresent
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  volumes:
  - name: docker-config
    emptyDir: {}
"""
    }
  }

  environment {
    AWS_REGION = "ap-south-1"
    ACCOUNT_ID = "433193941125"
    ECR_REGISTRY = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

    BACKEND_IMAGE  = "${ECR_REGISTRY}/movie-backend"
    FRONTEND_IMAGE = "${ECR_REGISTRY}/movie-frontend"
    MODEL_IMAGE    = "${ECR_REGISTRY}/movie-model"

    IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {

    stage("Checkout") {
      steps {
        checkout scm
      }
    }

    stage("Build & Push Backend") {
      steps {
        container("kaniko") {
          sh """
          /kaniko/executor \
            --dockerfile=backend/Dockerfile \
            --context=backend \
            --destination=${BACKEND_IMAGE}:${IMAGE_TAG} \
            --destination=${BACKEND_IMAGE}:latest
          """
        }
      }
    }

    stage("Build & Push Frontend") {
      steps {
        container("kaniko") {
          sh """
          /kaniko/executor \
            --dockerfile=frontend/Dockerfile \
            --context=frontend \
            --destination=${FRONTEND_IMAGE}:${IMAGE_TAG} \
            --destination=${FRONTEND_IMAGE}:latest
          """
        }
      }
    }

    stage("Build & Push Model") {
      steps {
        container("kaniko") {
          sh """
          /kaniko/executor \
            --dockerfile=model/Dockerfile \
            --context=model \
            --destination=${MODEL_IMAGE}:${IMAGE_TAG} \
            --destination=${MODEL_IMAGE}:latest
          """
        }
      }
    }
  }

  post {
    success {
      echo "✅ Images built and pushed to ECR successfully"
    }
    failure {
      echo "❌ Build failed"
    }
  }
}
