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
    }

    stage('Push to ECR') {
    steps {
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws-creds'
        ]]) {

            sh '''
            aws ecr get-login-password --region ap-south-1 | \
            docker login --username AWS --password-stdin 484907501702.dkr.ecr.ap-south-1.amazonaws.com

            docker tag taskmanager:${BUILD_NUMBER} \
            484907501702.dkr.ecr.ap-south-1.amazonaws.com/taskmanager:${BUILD_NUMBER}

            docker push \
            484907501702.dkr.ecr.ap-south-1.amazonaws.com/taskmanager:${BUILD_NUMBER}
            '''
        }
    }
}

    post {
        always {
            echo 'Pipeline execution finished.'
        }
    }
}
