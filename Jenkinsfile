pipeline {
    agent any

    environment {
        AWSCRAD = '45b042f8-edcf-437e-8cc8-cad41343e56e'
        REGION = 'us-east-1'
        ACCOUNTID = '992382545251'
        IMAGENAME = 'calcapp'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${env.IMAGENAME} ."
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([
                    aws(credentialsId: env.AWSCRAD, region: env.REGION)
                ]) {
                    sh "aws ecr get-login-password --region ${env.REGION} | docker login --username AWS --password-stdin ${env.ACCOUNTID}.dkr.ecr.${env.REGION}.amazonaws.com"
                    sh "docker push ${env.ACCOUNTID}.dkr.ecr.${env.REGION}.amazonaws.com/${env.IMAGENAME}:latest"
                }
            }
        }
        stage('Deploy Docker Container') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: '<credentials_id>', keyFileVariable: 'SSH_KEY_FILE')
                ]) {
                    sh 'scp -o StrictHostKeyChecking=no -i $SSH_KEY_FILE Dockerrun.aws.json ec2-user@<ec2_instance_ip>:/home/ec2-user/'
                    sh 'ssh -o StrictHostKeyChecking=no -i $SSH_KEY_FILE ec2-user@<ec2_instance_ip> "docker stop <container_name> || true"'
                    sh 'ssh -o StrictHostKeyChecking=no -i $SSH_KEY_FILE ec2-user@<ec2_instance_ip> "docker rm <container_name> || true"'
                    sh 'ssh -o StrictHostKeyChecking=no -i $SSH_KEY_FILE ec2-user@<ec2_instance_ip> "docker run -d --name <container_name> -p 80:8080 ${env.ACCOUNTID}.dkr.ecr.${env.REGION}.amazonaws.com/${env.IMAGENAME}:latest"'
                }
            }
        }
    }
}


