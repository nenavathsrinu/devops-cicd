pipeline {
  agent any

  environment {
    AWS_REGION = 'ap-south-1'
    ECR_REPO = '<your-account-id>.dkr.ecr.ap-south-1.amazonaws.com/my-node-app'
    IMAGE_TAG = "v1.${BUILD_NUMBER}"
  }

  stages {
    stage('Clone Repository') {
      steps {
        git url: 'https://github.com/nenavathsrinu/devops-cicd.git', branch: 'main'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t my-node-app .'
        sh 'docker tag my-node-app:latest $ECR_REPO:$IMAGE_TAG'
      }
    }

    stage('Login to AWS ECR') {
      steps {
        sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO'
      }
    }

    stage('Push to ECR') {
      steps {
        sh 'docker push $ECR_REPO:$IMAGE_TAG'
      }
    }
  }

  post {
    success {
      echo "Docker image pushed to ECR: $ECR_REPO:$IMAGE_TAG"
    }
    failure {
      echo "Build failed"
    }
  }
}
