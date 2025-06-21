pipeline {
  agent any

  environment {
    AWS_REGION = 'ap-south-1'
    ACCOUNT_ID = '977076879209' // 🔁 Replace with your actual AWS Account ID
    REPO_NAME = 'my-node-app'
    IMAGE_TAG = "v1.${BUILD_NUMBER}"
    ECR_REPO = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
  }

  stages {
    stage('Clone Repository') {
      steps {
        git url: 'https://github.com/nenavathsrinu/devops-cicd-ecs.git', branch: 'main'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          echo "🔧 Building Docker image..."
          docker build -t my-node-app .
          docker tag my-node-app:latest $ECR_REPO:$IMAGE_TAG
        '''
      }
    }

    stage('Login to AWS ECR') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred']]) {
          sh '''
            echo "🔐 Logging into AWS ECR..."
            aws ecr get-login-password --region $AWS_REGION | \
            docker login --username AWS --password-stdin $ECR_REPO
          '''
        }
      }
    }

    stage('Push to ECR') {
      steps {
        sh '''
          echo "📤 Pushing Docker image to ECR..."
          docker push $ECR_REPO:$IMAGE_TAG
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Image pushed: $ECR_REPO:$IMAGE_TAG"
    }
    failure {
      echo "❌ Build or push failed"
    }
  }
}
