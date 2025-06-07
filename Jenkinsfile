pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        DOCKER_HOST = 'tcp://host.docker.internal:2375'
        DOCKER_TLS_VERIFY = '0'
        DOCKER_CERT_PATH = ''
        IMAGE_TAG = "nodeimage${env.BUILD_NUMBER}"
    }
    stages {
        stage('Debug Environment') {
            steps {
                sh '''
                    echo "Kiểm tra môi trường..."
                    unset DOCKER_TLS_VERIFY
                    unset DOCKER_CERT_PATH
                    export DOCKER_TLS_VERIFY=0
                    # Xóa thư mục cấu hình Docker nếu tồn tại
                    rm -rf /var/jenkins_home/.docker || echo "Không thể xóa thư mục cấu hình Docker"
                    # Kiểm tra kết nối tới Docker daemon
                    curl -s http://host.docker.internal:2375/_ping || echo "Không thể kết nối tới Docker daemon qua HTTP"
                    DOCKER_HOST=tcp://host.docker.internal:2375 docker info || echo "Không thể kết nối tới Docker daemon"
                '''
            }
        }
        stage('Checkout Github') {
            steps {
                git branch: '18.0', credentialsId: 'github-credentials-id', url: 'https://github.com/owlgroup/odoo.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh '''
                    unset DOCKER_TLS_VERIFY
                    unset DOCKER_CERT_PATH
                    export DOCKER_TLS_VERIFY=0
                    rm -rf /var/jenkins_home/.docker || echo "Không thể xóa thư mục cấu hình Docker"
                    DOCKER_HOST=tcp://host.docker.internal:2375 docker build -t "$IMAGE_TAG" .
                '''
            }
        }
        stage('Deploy to Localhost') {
            steps {
                sh '''
                    unset DOCKER_TLS_VERIFY
                    unset DOCKER_CERT_PATH
                    export DOCKER_TLS_VERIFY=0
                    rm -rf /var/jenkins_home/.docker || echo "Không thể xóa thư mục cấu hình Docker"
                    DOCKER_HOST=tcp://host.docker.internal:2375 docker stop odoo-website || true
                    DOCKER_HOST=tcp://host.docker.internal:2375 docker rm odoo-website || true
                    DOCKER_HOST=tcp://host.docker.internal:2375 docker run -d --name odoo-website -p 8069:8069 "$IMAGE_TAG"
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