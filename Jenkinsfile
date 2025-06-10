pipeline {
    agent any
    environment {
        IMAGE_NAME = "odoo-website" // Tên image cố định
        IMAGE_TAG = "${env.IMAGE_NAME}:${env.BUILD_NUMBER}" // Tag image với build number
        DOCKER_REGISTRY = 'https://index.docker.io/v1/' // Registry URL
        DOCKER_CREDENTIALS_ID = 'docker-key' // ID của credentials trong Jenkins
        // Nếu Docker daemon chạy trên cùng máy, không cần DOCKER_HOST
        // DOCKER_HOST = 'tcp://192.168.1.117:2375' // Chỉ dùng nếu cần
    }
    stages {
        stage('Check Docker Environment') {
            steps {
                script {
                    try {
                        sh 'docker info --format "{{.ServerVersion}}"'
                        echo "Docker daemon is accessible"
                    } catch (Exception e) {
                        error "Docker daemon not accessible: ${e.message}"
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh "docker build -t ${IMAGE_TAG} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry(credentialsId: "${DOCKER_CREDENTIALS_ID}", url: "${DOCKER_REGISTRY}") {
                    sh "docker push ${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy to Localhost') {
            steps {
                script {
                    try {
                        // Dừng và xóa container cũ nếu có
                        sh '''
                            docker stop odoo-website || true
                            docker rm odoo-website || true
                        '''
                        // Chạy container mới
                        sh "docker run -d --name odoo-website -p 8069:8069 ${IMAGE_TAG}"
                        // Kiểm tra container có chạy không
                        sleep 5 // Đợi vài giây để container khởi động
                        sh 'docker ps --filter "name=odoo-website" --format "{{.Names}}" | grep odoo-website'
                        echo "Container odoo-website is running"
                    } catch (Exception e) {
                        error "Deployment failed: ${e.message}"
                    }
                }
            }
        }
    }
    post {
        success {
            echo '✅ Xây dựng và triển khai thành công!'
            // Có thể thêm thông báo qua Slack, email, v.v.
            // Ví dụ: slackSend(channel: '#devops', message: "Build #${env.BUILD_NUMBER} deployed successfully!")
        }
        failure {
            echo '❌ Xây dựng hoặc triển khai thất bại.'
            // Có thể thêm thông báo lỗi
            // Ví dụ: slackSend(channel: '#devops', message: "Build #${env.BUILD_NUMBER} failed!")
        }
        always {
            // Dọn dẹp các image không cần thiết để tiết kiệm không gian
            sh "docker image prune -f"
        }
    }
}