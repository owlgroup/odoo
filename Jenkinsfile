pipeline {
    agent any
    environment {
        IMAGE_TAG = "nodeimage${env.BUILD_NUMBER}"
        DOCKER_HOST = "tcp://localhost:2375"
    }
    stages {
        stage('Check Docker Environment') {
            steps {
                bat '''
                    echo "Checking Docker environment..."
                    docker info || echo "Docker daemon not accessible"
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                bat '''
                    docker build -t "%IMAGE_TAG%" .
                '''
            }
        }
        stage('Deploy to Localhost') {
            steps {
                bat '''
                    docker stop odoo-website || exit 0
                    docker rm odoo-website || exit 0
                    docker run -d --name odoo-website -p 8069:8069 "%IMAGE_TAG%"
                '''
            }
        }
    }
    post {
        success {
            echo '✅ Xây dựng và triển khai thành công!'
        }
        failure {
            echo '❌ Xây dựng hoặc triển khai thất bại.'
        }
    }
}