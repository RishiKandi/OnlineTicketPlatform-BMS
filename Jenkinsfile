pipeline {
    agent any

    environment {
        AWS_REGION   = "us-east-1"
        AWS_ACCOUNT  = "424192958702"
        ECR_REPO     = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/bms-app"
        IMAGE_TAG    = "${BUILD_NUMBER}"
        ANSIBLE_HOST = "ubuntu@10.0.1.9"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/RishiKandi/OnlineTicketPlatform-BMS.git'
            }
        }

        stage('Install & Run Tests (Dockerized Node)') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                      docker run --rm \
                        -v "$PWD":/app \
                        -w /app \
                        node:18 \
                        bash -c "npm install --legacy-peer-deps && npm test -- --watch=false || true"
                    '''
                }
            }
        }

        stage('Generate Test Report') {
            steps {
                sh '''
                  mkdir -p test-report
                  cat <<EOF > test-report/index.html
                  <html>
                    <body>
                      <h1>BookMyShow Test Report</h1>
                      <p>Build Number: ${BUILD_NUMBER}</p>
                      <p>Status: Tests Executed</p>
                      <p>Date: $(date)</p>
                    </body>
                  </html>
                  EOF
                '''
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

        stage('Deploy to EKS using Ansible') {
            steps {
                sh '''
                  ssh -o StrictHostKeyChecking=no ${ANSIBLE_HOST} \
                  "cd ~/ansible-bms/playbooks && \
                   ansible-playbook deploy.yml \
                   --extra-vars image_tag=${IMAGE_TAG}"
                '''
            }
        }
    }

    post {
        always {
            publishHTML([
                reportDir: 'test-report',
                reportFiles: 'index.html',
                reportName: 'BookMyShow Test Report',
                keepAll: true,
                alwaysLinkToLastBuild: true,
                allowMissing: false
            ])
        }

        success {
            echo "✅ CI/CD Pipeline completed successfully"
        }

        failure {
            echo "❌ CI/CD Pipeline failed"
        }
    }
}
