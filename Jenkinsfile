pipeline {
    agent any
   
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '905418472653.dkr.ecr.us-east-1.amazonaws.com/practical-devops'
    }
   
    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    // Build Docker images
                    docker.build("backend:${env.BUILD_NUMBER}", "-f ${env.WORKSPACE}/src/backend/Dockerfile .")
                    docker.build("frontend:${env.BUILD_NUMBER}", "-f ${env.WORKSPACE}/src/frontend/Dockerfile .")
                }
            }
        }
       
        stage('Tag Docker Image') {
            steps {
                script {
                    // Tag Docker image using Docker command directly
                    sh "docker tag backend:${env.BUILD_NUMBER} $ECR_REPO:${env.BUILD_NUMBER}"
                    sh "docker tag frontend:${env.BUILD_NUMBER} $ECR_REPO:${env.BUILD_NUMBER}"
                }
            }
        }


        stage('Push to ECR') {
            steps {
                script {
                    // Authenticate Docker client to ECR using AWS CLI
                    withCredentials([aws(credentialsId: '905418472653', region: AWS_REGION)]) {
                        sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO"
                    }
                   
                    // Push Docker image to ECR
                    sh "docker push $ECR_REPO:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('ManifestUpdate') {
            steps {
                script {
                    build job: 'eks-pipeline', parameters: [string(name: 'IMAGE_TAG', value: env.BUILD_NUMBER)]
                }
            }
        }
    }
}
