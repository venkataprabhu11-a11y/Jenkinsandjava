
pipeline {
    agent any

    environment {
        GIT_REPO      = 'https://github.com/venkataprabhu11-a11y/Jenkinsandjava.git'
        AWS_REGION    = 'ap-south-2'
        ECR_REPO_NAME = 'sample-project1'
        ECR_REGISTRY  = '492715478536.dkr.ecr.ap-south-2.amazonaws.com'
        IMAGE_TAG     = 'latest'
        IMAGE_URI     = "${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Install AWS CLI') {
            steps {
                sh '''
                    set -e
                    echo "Installing AWS CLI..."

                    mkdir -p "$HOME/.local/bin"

                    if ! command -v curl >/dev/null 2>&1 || \
                       ! command -v unzip >/dev/null 2>&1; then
                        echo "ERROR: curl and unzip must already be installed."
                        exit 1
                    fi

                    curl -fsSL \
                        "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
                        -o "$HOME/awscliv2.zip"

                    rm -rf "$HOME/aws" "$HOME/.local/aws-cli"
                    unzip -q "$HOME/awscliv2.zip" -d "$HOME"

                    "$HOME/aws/install" \
                        --install-dir "$HOME/.local/aws-cli" \
                        --bin-dir "$HOME/.local/bin"

                    rm -rf "$HOME/aws" "$HOME/awscliv2.zip"

                    export PATH="$HOME/.local/bin:$PATH"
                    aws --version
                '''
            }
        }

        stage('Configure AWS Credentials') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        set -e
                        export PATH="$HOME/.local/bin:$PATH"
                        export AWS_DEFAULT_REGION="$AWS_REGION"

                        echo "Checking AWS credentials..."
                        aws sts get-caller-identity
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
                sh '''
                    set -e
                    echo "Building Java application..."
                    mvn clean -B -Denforcer.skip=true package
                '''
            }
        }

        
stage('Login to AWS ECR') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'aws-credentials',
                usernameVariable: 'AWS_ACCESS_KEY_ID',
                passwordVariable: 'AWS_SECRET_ACCESS_KEY'
            )
        ]) {
            sh '''
                set -eu
                export PATH="$HOME/.local/bin:$PATH"
                export AWS_DEFAULT_REGION="$AWS_REGION"

                echo "Checking AWS identity..."
                aws sts get-caller-identity

                echo "Checking ECR authorization..."
                aws ecr get-login-password \
                    --region "$AWS_REGION" \
                    > "$WORKSPACE@tmp/ecr-password"

                echo "Logging into private AWS ECR..."
                docker login \
                    --username AWS \
                    --password-stdin "$ECR_REGISTRY" \
                    < "$WORKSPACE@tmp/ecr-password"

                rm -f "$WORKSPACE@tmp/ecr-password"
            '''
        }
    }
}


        stage('Build Docker Image') {
            steps {
                sh '''
                    set -e
                    echo "Building Docker image..."
                    docker build -t "$IMAGE_URI" .
                '''
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh '''
                    set -e
                    echo "Pushing Docker image to ECR..."
                    docker push "$IMAGE_URI"
                '''
            }
        }
    }

    post {
        success {
            echo 'Docker image pushed to ECR successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the console output above.'
        }

        always {
            sh 'docker logout "$ECR_REGISTRY" || true'
        }
    }
}
