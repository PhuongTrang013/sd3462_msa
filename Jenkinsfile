pipeline {
    agent any
   
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '905418472653.dkr.ecr.us-east-1.amazonaws.com/practical-devops'
        KUBECONFIG = "/var/lib/jenkins/.kube/config" //locate in jenkins EC2 serevr
    }
   
    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    // Build Docker images
                    def backend = docker.build("backend:${env.GIT_COMMIT}", "-f ${env.WORKSPACE}/src/backend/Dockerfile .")
                    def frontend = docker.build("frontend:${env.GIT_COMMIT}", "-f ${env.WORKSPACE}/src/frontend/Dockerfile .")
                }
            }
        }
       
        stage('Tag Docker Image') {
            steps {
                script {
                    // Tag Docker image using Docker command directly
                    sh "docker tag backend:${env.GIT_COMMIT} $ECR_REPO:${env.GIT_COMMIT}"
                    sh "docker tag frontend:${env.GIT_COMMIT} $ECR_REPO:${env.GIT_COMMIT}"
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
                    docker manifest create $ECR_REPO $ECR_REPO:backend:${env.GIT_COMMIT} $ECR_REPO:frontend:${env.GIT_COMMIT}
                    docker manifest inspect $ECR_REPO
                    docker manifest push $ECR_REPO
                }
            }
        }
    }
}
