pipeline {
    agent{
         node{
             label "Slave-1"
             customWorkspace "/home/jenkins/dexter"
         }
    }
    environment{
        JAVA_HOME="/usr/lib/jvm/java-17-amazon-corretto.x86_64"
        PATH="$PATH:$JAVA_HOME/bin:/opt/apache-maven/bin"
    }
    parameters { 
        string(name: 'REPO_NAME', defaultValue: '', description: 'Provide the ECR Repository Name for Application Image')
        string(name: 'TAG_NUMBER', defaultValue: '', description: 'Provide the TAG NUMBER')
    }
    stages {
        stage('git checkout source-code') {
            steps {
                cleanWs()
                checkout scmGit(branches: [[name: 'main']], extensions: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/singhritesh85/BankApp-Docker-the-Serverless-way.git']])
            }
        }
        stage('git Checkout terraform script') {
            steps {
                dir('/home/jenkins/dexter/tetra/') {
                    cleanWs()
                    checkout scmGit(branches: [[name: 'main']], extensions: [], userRemoteConfigs: [[credentialsId: 'GitHub-PAT', url: 'https://github.com/singhritesh85/terraform-docker-the-serverless-way.git']])
                }
            }
        }
        stage('Build Source Code') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Build Docker Image') {
            steps{
                sh 'docker system prune -f --all'
                sh 'docker build -t ${REPO_NAME}:${TAG_NUMBER} -f Dockerfile-Project-1 .'
                sh 'trivy image --exit-code 0 --severity MEDIUM,HIGH ${REPO_NAME}:${TAG_NUMBER}'
                sh 'trivy image  --exit-code 1 --severity CRITICAL ${REPO_NAME}:${TAG_NUMBER}'
                sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 027330342406.dkr.ecr.us-east-2.amazonaws.com'
                sh 'docker push ${REPO_NAME}:${TAG_NUMBER}'
            }
        }
        stage('Terraform Init') {
            steps {
                dir('/home/jenkins/dexter/tetra/terraform-ecs-task-definition-service-ecr-alb/main/') {
                    sh 'terraform init'
                }
            }
        }
        stage('Terraform Plan') {
            steps {
                dir('/home/jenkins/dexter/tetra/terraform-ecs-task-definition-service-ecr-alb/main/') {
                    sh 'terraform plan'
                }
            }
        }
        stage('Terraform Apply') {
            steps {
                dir('/home/jenkins/dexter/tetra/terraform-ecs-task-definition-service-ecr-alb/main/') {
                    sh 'terraform apply -auto-approve -var TAG_NUMBER=${TAG_NUMBER}'
                }
            }
        }
    }
}
