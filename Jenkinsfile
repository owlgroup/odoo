pipeline {
    agent any
    environment {
        IMAGE_TAG = "nodeimage${env.BUILD_NUMBER}"
        // Sử dụng giá trị phù hợp dựa trên vị trí của Docker daemon
        DOCKER_HOST = "tcp://192.168.1.117:2375" // Nếu Docker trên cùng máy với Jenkins
        // Hoặc DOCKER_HOST = "tcp://<correct-ip>:2375" // Nếu Docker trên máy khác
    }
    stages {
        stage('Check Docker Environment') {
            steps {
                sh '''
                    echo "Checking Docker environment..."
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