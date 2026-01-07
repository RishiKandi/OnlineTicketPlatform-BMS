pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "424192958702.dkr.ecr.us-east-1.amazonaws.com/bms-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        ANSIBLE_HOST = "ubuntu@10.0.1.9"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/RishiKandi/OnlineTicketPlatform-BMS.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                    docker build -t bms-app:${IMAGE_TAG} .
                    docker tag bms-app:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} \
                | docker login --username AWS --password-stdin ${ECR_REPO}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker push ${ECR_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Trigger Ansible Deployment') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ${ANSIBLE_HOST} \
                "cd ~/ansible-bms/playbooks && ansible-playbook deploy-bms.yaml \
                --extra-vars image_tag=${IMAGE_TAG}"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
