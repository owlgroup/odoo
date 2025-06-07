pipeline {
    agent any
    environment {
        IMAGE_TAG = "nodeimage${env.BUILD_NUMBER}"
        DOCKER_HOST = "unix:///var/run/docker.sock"
    }
    stages {
        stage('Check Docker Environment') {
            steps {
                sh '''
                    echo "DOCKER_HOST: $DOCKER_HOST"
                    docker info || echo "Docker daemon not accessible"
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t "$IMAGE_TAG" .
                '''
            }
        }
        stage('Deploy to Localhost') {
            steps {
                sh '''
                    docker stop odoo-website || true
                    docker rm odoo-website || true
                    docker run -d --name odoo-website -p 8069:8069 "$IMAGE_TAG"
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