
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
                    docker build -t $IMAGE_NAME:$BUILD_NUMBER .
                '''
            }
        }

        stage('Docker Tag') {
            steps {
                sh '''
                    docker tag $IMAGE_NAME:$BUILD_NUMBER \
                    $ECR_REGISTRY/$IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    set +x
                    aws ecr get-login-password --region "$AWS_REGION" | \
                    docker login --username AWS --password-stdin "$ECR_REGISTRY"
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    docker push $ECR_REGISTRY/$IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }
    }

    post {
        success {
            echo 'Docker image built and pushed to AWS ECR successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}
