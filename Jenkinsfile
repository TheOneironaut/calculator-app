pipeline {
    agent any
    
    environment {
        // AWS Configuration
        AWSCRED     = 'ec443a4a-c71a-4b3e-bf6b-2a378f47d76a'
        REGION      = 'us-east-1'
        ACCOUNTID   = '992382545251'
        
        // Docker Image Configuration
        IMAGENAME   = 'amitay-jenk'  // עודכן להתאים ל-repository הקיים ב-ECR
        IMAGE_TAG   = 'latest'
        
        // SSH Configuration  
        SSHCRED     = 'deb05777-3bce-4ec0-8432-68f952f7528e'
        EC2_HOST    = '52.90.190.67' 
        EC2_USER    = 'ec2-user'
        
        // Derived Variables
        ECR_REGISTRY = "${ACCOUNTID}.dkr.ecr.${REGION}.amazonaws.com"
        FULL_IMAGE_NAME = "${ECR_REGISTRY}/${IMAGENAME}:${IMAGE_TAG}"
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${env.IMAGENAME}:${env.IMAGE_TAG} ."
                sh "docker tag ${env.IMAGENAME}:${env.IMAGE_TAG} ${env.FULL_IMAGE_NAME}"
            }
        }
        
        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([
                    aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                        credentialsId: "${env.AWSCRED}", 
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh """
                        aws ecr get-login-password --region ${env.REGION} \\
                          | docker login --username AWS --password-stdin ${env.ECR_REGISTRY}
                        docker push ${env.FULL_IMAGE_NAME}
                    """
                }
            }
        }
        
        stage('Deploy Docker Container') {
            when {
                expression {
                    "${env.GIT_BRANCH}" == 'origin/main'
                }
            }
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: "${env.SSHCRED}", keyFileVariable: 'SSH_KEY_FILE'),
                    aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "${env.AWSCRED}", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh """
                        # הרצת פקודות מרוכבות על ה-EC2 instance
                        ssh -o StrictHostKeyChecking=no -i \$SSH_KEY_FILE ${env.EC2_USER}@${env.EC2_HOST} "
                            export AWS_ACCESS_KEY_ID=\$AWS_ACCESS_KEY_ID && \\
                            export AWS_SECRET_ACCESS_KEY=\$AWS_SECRET_ACCESS_KEY && \\
                            export AWS_DEFAULT_REGION=${env.REGION} && \\
                            aws ecr get-login-password --region ${env.REGION} | sudo docker login --username AWS --password-stdin ${env.ECR_REGISTRY} && \\
                            sudo docker pull ${env.FULL_IMAGE_NAME} && \\
                            sudo docker stop ${env.IMAGENAME} || true && \\
                            sudo docker rm ${env.IMAGENAME} || true && \\
                            sudo docker run -d --name ${env.IMAGENAME} -p 80:8080 ${env.FULL_IMAGE_NAME}
                        "
                    """
                }
            }
        }

    }
}




