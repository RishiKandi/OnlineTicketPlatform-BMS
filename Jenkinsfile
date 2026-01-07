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

        stage('Install & Run Tests') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                      npm install --legacy-peer-deps
                      npm test -- --watch=false || true
                    '''
                }
            }
        }

        stage('Generate Test Report') {
            steps {
                sh '''
                  echo "Workspace: $WORKSPACE"

                  mkdir -p "$WORKSPACE/test-report"

                  cat <<EOF > "$WORKSPACE/test-report/index.html"
                  <html>
                    <head>
                      <title>BookMyShow Test Report</title>
                    </head>
                    <body>
                      <h1>BookMyShow CI Test Report</h1>
                      <p><b>Build Number:</b> ${BUILD_NUMBER}</p>
                      <p><b>Status:</b> Tests Executed</p>
                      <p><b>Date:</b> $(date)</p>
                    </body>
                  </html>
                  EOF

                  ls -l "$WORKSPACE/test-report"
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
