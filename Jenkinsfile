pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        IMAGE_NAME = 'taskmanager'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Secret Scan') {
            steps {
                sh 'gitleaks detect --source . -v'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build \
                -t ${IMAGE_NAME}:${IMAGE_TAG} \
                -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        // Add ECR stage later

        // Add Helm stage later

        // ArgoCD will deploy automatically
    }

    post {
        always {
            echo 'Pipeline execution finished.'
        }

        success {
            echo 'Pipeline succeeded!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
