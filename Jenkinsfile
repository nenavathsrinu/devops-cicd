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
      script {
        def taskDef = """
        {
          "family": "${env.REPO_NAME}-task",
          "networkMode": "awsvpc",
          "executionRoleArn": "arn:aws:iam::${env.ACCOUNT_ID}:role/${env.TASK_ROLE}",
          "containerDefinitions": [{
            "name": "${env.CONTAINER}",
            "image": "${env.ECR_REPO}:${env.IMAGE_TAG}",
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
        """

        writeFile file: 'taskdef.json', text: taskDef

        sh '''
          echo "⚙️ Registering new ECS Task Definition..."
          aws ecs register-task-definition --cli-input-json file://taskdef.json

          echo "🔄 Updating ECS service..."
          aws ecs update-service \
            --cluster $CLUSTER \
            --service $SERVICE \
            --task-definition ${REPO_NAME}-task
        '''
      }
    }
  }
}
