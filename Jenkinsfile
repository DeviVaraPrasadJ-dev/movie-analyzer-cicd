pipeline {
  agent {
    kubernetes {
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command: ["sh", "-c", "sleep 999999"]
    tty: true
"""
    }
  }

  environment {
    AWS_REGION   = "ap-south-1"
    ECR_ACCOUNT  = "433193941125"
    ECR_REGISTRY = "\${ECR_ACCOUNT}.dkr.ecr.\${AWS_REGION}.amazonaws.com"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Backend Image') {
      steps {
        container('kaniko') {
          sh '''
            ls -l backend
          /kaniko/executor \
            --context=dir://${WORKSPACE}/backend \
            --dockerfile=Dockerfile \
            --destination=${ECR_REGISTRY}/movie-backend:${BUILD_NUMBER} \
            --verbosity=info

          '''
        }
      }
    }
  }
}
