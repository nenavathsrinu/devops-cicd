pipeline {
  agent any

  environment {
    AWS_REGION = 'ap-south-1'
    ACCOUNT_ID = '977076879209'  // 🔁 Replace with your AWS Account ID
    REPO_NAME  = 'my-node-app'
    IMAGE_TAG  = "v1.${BUILD_NUMBER}"
    CLUSTER    = 'devops-cluster'
    SERVICE    = 'nodejs-service'
    TASK_ROLE  = 'ecsTaskExecutionRole'
    CONTAINER  = 'nodejs-container'
    ECR_REPO   = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/nenavathsrinu/devops-cicd.git', branch: 'main'
      }
    }

    stage('Build & Push to ECR') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred']]) {
          sh '''
            docker build -t $REPO_NAME .
            docker tag $REPO_NAME:latest $ECR_REPO:$IMAGE_TAG
            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
            docker push $ECR_REPO:$IMAGE_TAG
          '''
        }
      }
    }

    stage('Deploy to ECS Fargate') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred']]) {
          sh '''
            echo "⚙️ Registering new task definition..."

            TASK_DEF=$(cat <<EOF
            {
              "family": "$REPO_NAME-task",
              "networkMode": "awsvpc",
              "executionRoleArn": "arn:aws:iam::$ACCOUNT_ID:role/$TASK_ROLE",
              "containerDefinitions": [{
                "name": "$CONTAINER",
                "image": "$ECR_REPO:$IMAGE_TAG",
                "portMappings": [{
                  "containerPort": 3000,
                  "hostPort": 3000,
                  "protocol": "tcp"
                }],
                "essential": true
              }],
              "requiresCompatibilities": ["FARGATE"],
              "cpu": "256",
              "memory": "512"
            }
            EOF
            )

            echo "$TASK_DEF" > taskdef.json

            aws ecs register-task-definition --cli-input-json file://taskdef.json

            echo "🔄 Updating ECS service..."

            aws ecs update-service \
              --cluster $CLUSTER \
              --service $SERVICE \
              --task-definition $REPO_NAME-task
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ Deployment to ECS complete: $ECR_REPO:$IMAGE_TAG"
    }
    failure {
      echo "❌ Deployment failed"
    }
  }
}
