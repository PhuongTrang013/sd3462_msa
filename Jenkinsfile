pipeline {
    agent any
   
    environment {
        AWS_REGION = 'us-east-1'
        AWS_CREDENTIALS_ID = '905418472653'
        ECR_REPO = '905418472653.dkr.ecr.us-east-1.amazonaws.com/practical-devops'
        IMAGE_TAG = "${env.BUILD_NUMBER}-${env.BUILD_ID}"  // Unique image tag
    }
   
    stages {
        stage('Backend: Build/Tag Docker Image') {
            steps {
                script {
                    // Build Docker images
                    docker.build("backend:${IMAGE_TAG}", "-f ${env.WORKSPACE}/src/backend/Dockerfile .")

                    // Tag Docker image using Docker command directly
                    sh "docker tag backend:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}"
                   
                }
            }
        }
       
        stage('Frontend: Build/Tag Docker Image') {
            steps {
                script {
                    // Build Docker images
                    docker.build("frontend:${IMAGE_TAG}", "-f ${env.WORKSPACE}/src/frontend/Dockerfile .")
                    
                    // Tag Docker image using Docker command directly
                    sh "docker tag frontend:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }


        stage('Push to ECR') {
            steps {
                script {
                    // Authenticate Docker client to ECR using AWS CLI
                    withCredentials([aws(credentialsId: "${AWS_CREDENTIALS_ID}", region: AWS_REGION)]) {
                        sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO"
                    }
                   
                    // Push Docker image to ECR
                    sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }

        stage('Trigger Manifest Update') {
            steps {
                script {
                    build job: 'eks-pipeline', parameters: [string(name: 'IMAGE_TAG', value: IMAGE_TAG)]
                }
            }
        }
    }
}
