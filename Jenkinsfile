pipeline {
    agent any
    environment {
        DOCKER_HOST = 'tcp://localhost:2375'
        DOCKER_TLS_VERIFY = '0'
        DOCKER_CERT_PATH = ''
        IMAGE_TAG = "nodeimage${env.BUILD_NUMBER}"
    }
    stages {
        stage('Build Docker Image') {
            steps {
                sh '''
                    unset DOCKER_TLS_VERIFY
                    unset DOCKER_CERT_PATH
                    export DOCKER_TLS_VERIFY=0
                    # Tạo thư mục cấu hình tạm thời để tránh lỗi chứng chỉ
                    mkdir -p /tmp/docker-config || true
                    export DOCKER_CONFIG=/tmp/docker-config
                    DOCKER_HOST=tcp://localhost:2375 docker build -t "$IMAGE_TAG" .
                '''
            }
        }
        stage('Deploy to Localhost') {
            steps {
                sh '''
                    unset DOCKER_TLS_VERIFY
                    unset DOCKER_CERT_PATH
                    export DOCKER_TLS_VERIFY=0
                    mkdir -p /tmp/docker-config || true
                    export DOCKER_CONFIG=/tmp/docker-config
                    DOCKER_HOST=tcp://localhost:2375 docker stop odoo-website || true
                    DOCKER_HOST=tcp://localhost:2375 docker rm odoo-website || true
                    DOCKER_HOST=tcp://localhost:2375 docker run -d --name odoo-website -p 8069:8069 "$IMAGE_TAG"
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