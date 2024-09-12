pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS-ACCESS-KEY')
        AWS_SECRET_ACCESS_KEY = credentials('AWS-SECRET-KEY')
    }
    stages {
        stage('Clone and Build Project') {
            steps {
                script {
                    // Generate a timestamp (without special characters)
                    env.TIMESTAMP = sh(returnStdout: true, script: 'date +%Y-%m-%d-%H-%M-%S').trim()

                    // Clone Git repository
                    git 'https://github.com/abhis2024/star-agile-banking-finance_abhis2024.git'

                    // Build the project using Maven
                    sh 'mvn clean package'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image with the timestamp tag
                    sh "docker build -t abhis2024/finance_app:${TIMESTAMP} ."

                    // List Docker images to verify
                    sh 'docker images'
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER-CREDS', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    // Login to Docker Hub
                    sh "echo $PASS | docker login -u $USER --password-stdin"

                    // Push the Docker image with the timestamp tag
                    sh "docker push abhis2024/finance_app:${TIMESTAMP}"
                }
            }
        }
        
        stage('Terraform Init & Destroy for Test Workspace') {
            steps {
                script {
                    // Switch to or create the test workspace
                    sh '''
                    #!/bin/bash
                    terraform workspace select test || terraform workspace new test
                    terraform init
                    terraform plan
                    terraform destroy -auto-approve
                    '''
                }
            }
        }
        stage('Terraform Apply for Test Workspace') {
            steps {
                script {
                    // Apply the Terraform plan for the test environment
                    sh 'terraform apply -auto-approve'
                }
            }
        }
        
        stage('Terraform Init & Apply for Prod Workspace') {
            when {
                expression { return currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                script {
                    // Switch to or create the prod workspace
                    sh '''
                    terraform workspace select prod || terraform workspace new prod
                    terraform init
                    if terraform state show aws_key_pair.example 2>/dev/null; then
                        echo "Key pair already exists in the prod workspace"
                    else
                        terraform import aws_key_pair.example project-key || echo "Key pair already imported"
                    fi
                    terraform destroy -auto-approve
                    terraform apply -auto-approve
                    '''
                }
            }
        }
    }
}

