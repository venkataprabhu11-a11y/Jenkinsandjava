
pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/venkataprabhu11-a11y/Jenkinsandjava.git'
        AWS_REGION = 'ap-south-2'
        ECR_REPO_NAME = 'sample-project1'
        ECR_PUBLIC_REPO_URI = '492715478536.dkr.ecr.ap-south-2.amazonaws.com/sample-project1'
        IMAGE_TAG = 'latest'
        AWS_ACCOUNT_ID = '492715478536'
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
    }

    stages {
        stage('Install AWS CLI') {
            steps {
                script {
                    sh '''
                        set -e
                        echo "Installing AWS CLI..."
                        sudo apt update && sudo apt install -y unzip curl

                        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                        rm -rf aws
                        unzip -q awscliv2.zip
                        sudo ./aws/install --update
                        aws --version
                    '''
                }
            }
        }

             stage('Configure AWS Credentials') {
    steps {
        script {
            sh '''
            echo "Setting up AWS credentials for Jenkins..."
            mkdir -p /var/lib/jenkins/.aws
            echo "[default]" > /var/lib/jenkins/.aws/credentials
            echo "aws_access_key_id=AKIAWN26KB25BRBD26NF" >> /var/lib/jenkins/.aws/credentials
            echo "aws_secret_access_key=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" >> /var/lib/jenkins/.aws/credentials
            chown -R jenkins:jenkins /var/lib/jenkins/.aws
            '''
        }
    }
}


        stage('Clone Repository') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh '''
                        echo "Building Java application..."
                        mvn clean -B -Denforcer.skip=true package
                    '''
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh '''
                        echo "Logging into AWS ECR..."
                        aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        echo "Building Docker image..."
                        docker build -t ${IMAGE_URI} .
                    '''
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    sh '''
                        echo "Pushing Docker image to ECR..."
                        docker push ${IMAGE_URI}
                    '''
                }
            }
        }

      
    }
    
    post {
        success {
            echo "Docker image pushed to ECR successfully and deployed."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
