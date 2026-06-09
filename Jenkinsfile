pipeline {
    agent any

    environment {
        IMAGE_NAME = 'taskmanager'
        IMAGE_TAG = "${env.BUILD_ID}"
    }

    stages {
        stage('Secret Scan') {
            steps {
                // Assuming gitleaks is installed on the Jenkins agent
                sh 'gitleaks detect --source . -v'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image...'
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Deploy (Mock)') {
            steps {
                echo "Deploying ${IMAGE_NAME}:${IMAGE_TAG} to Kubernetes..."
                // In a real pipeline, you would use 'helm upgrade' or 'kubectl apply' here
                // sh "helm upgrade --install taskmanager ./helm/taskmanager-chart --set image.tag=${IMAGE_TAG}"
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution finished.'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed! Please check logs.'
        }
    }
}
