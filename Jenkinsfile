pipeline {
    agent any
   
    environment {
        AWS_REGION = 'us-east-1'
        AWS_CredentialsId = 'a61005e5-e4c9-42a7-939c-cef3e77e87ad'
        ECR_REPO = '905418472653.dkr.ecr.us-east-1.amazonaws.com/practical-devops'
        KUBECONFIG = "/var/lib/jenkins/.kube/config" //locate in jenkins EC2 serevr
    }
   
    stages {
        stage('Build Docker Images') {
            steps {
                sh 'docker build -f ${env.WORKSPACE}/src/backend/Dockerfile -t backend:${env.GIT_COMMIT} .'
                sh 'docker build -f ${env.WORKSPACE}/src/frontend/Dockerfile -t frontend:${env.GIT_COMMIT} .'
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
                    withCredentials([aws(credentialsId: AWS_CredentialsId, region: AWS_REGION)]) {
                        sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO"
                    }
                   
                    // Push Docker image to ECR
                    sh "docker push $ECR_REPO:${env.GIT_COMMIT}"
                }
            }
        }
    }
}
