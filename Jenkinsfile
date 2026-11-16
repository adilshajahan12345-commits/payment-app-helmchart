
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        AWS_REGION = 'ap-southeast-2'
        ECR_REGISTRY = '501634099654.dkr.ecr.ap-southeast-2.amazonaws.com'
        IMAGE_NAME = 'payment-app'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Tools and AWS Access') {
            steps {
                sh '''
                    docker --version
                    aws --version
                    aws sts get-caller-identity
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t \
                    "$IMAGE_NAME:$BUILD_NUMBER" .
                '''
            }
        }

        stage('Docker Tag') {
            steps {
                sh '''
                    docker tag \
                    "$IMAGE_NAME:$BUILD_NUMBER" \
                    "$ECR_REGISTRY/$IMAGE_NAME:$BUILD_NUMBER"
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    set +x
                    set -e

                    aws ecr get-login-password \
                        --region "$AWS_REGION" |
                    docker login \
                        --username AWS \
                        --password-stdin "$ECR_REGISTRY"
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    docker push \
                    "$ECR_REGISTRY/$IMAGE_NAME:$BUILD_NUMBER"
                '''
            }
        }

        stage('Trigger Deploy Job') {
            steps {
                echo "Starting deployment for image tag: ${env.BUILD_NUMBER}"

                build job: 'payment-app-deploy',
                    parameters: [
                        string(
                            name: 'IMAGE_TAG',
                            value: "${env.BUILD_NUMBER}"
                        )
                    ],
                    wait: true,
                    propagate: true
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
            echo 'Docker image pushed to ECR and deployed to Kubernetes.'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check the console output.'
        }
    }
}
