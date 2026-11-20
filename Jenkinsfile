
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        AWS_REGION = 'ap-southeast-2'
        REGISTRY = '501634099654.dkr.ecr.ap-southeast-2.amazonaws.com'
        IMAGE_NAME = 'payment-app'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check AWS') {
            steps {
                sh 'aws sts get-caller-identity'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t payment-app:$BUILD_NUMBER .'
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    set +x
                    set -e
                    set -o pipefail

                    aws ecr get-login-password --region "$AWS_REGION" |
                    docker login -u AWS --password-stdin "$REGISTRY"

                    docker tag payment-app:$BUILD_NUMBER \
                    "$REGISTRY/$IMAGE_NAME:$BUILD_NUMBER"

                    docker push "$REGISTRY/$IMAGE_NAME:$BUILD_NUMBER"
                '''
            }
        }

        stage('Helm Push') {
            steps {
                sh '''
                    set +x
                    set -e
                    set -o pipefail

                    helm package ./first-chart \
                    --version "0.1.$BUILD_NUMBER"

                    aws ecr get-login-password --region "$AWS_REGION" |
                    helm registry login -u AWS --password-stdin "$REGISTRY"

                    helm push "first-chart-0.1.$BUILD_NUMBER.tgz" \
                    "oci://$REGISTRY"
                '''
            }
        }

        stage('Trigger Deploy Job') {
            steps {
                build job: 'payment-app-deploy',
                    parameters: [
                        string(name: 'IMAGE_TAG', value: "${env.BUILD_NUMBER}"),
                        string(name: 'CHART_VERSION', value: "0.1.${env.BUILD_NUMBER}")
                    ],
                    wait: true,
                    propagate: true
            }
        }
    }

    post {
        success {
            echo 'Build and deployment successful!'
        }

        failure {
            echo 'Pipeline failed. Check console output.'
        }
    }
}
