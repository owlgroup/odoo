pipeline {
    agent any
    environment {
        IMAGE_NAME = "odoo-website" // Tên image cố định
        IMAGE_TAG = "${env.IMAGE_NAME}:${env.BUILD_NUMBER}" // Tag image với build number
        DOCKER_REGISTRY = 'https://index.docker.io/v1/' // Registry URL
        DOCKER_CREDENTIALS_ID = 'docker-hub' // ID của credentials trong Jenkins (đảm bảo khớp với cấu hình của bạn)
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
                    echo "Docker image ${IMAGE_TAG} built successfully"
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry(credentialsId: "${DOCKER_CREDENTIALS_ID}", url: "${DOCKER_REGISTRY}") {
                    sh "docker push ${IMAGE_TAG}"
                    echo "Docker image ${IMAGE_TAG} pushed to DockerHub"
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
                            docker stop postgres-odoo || true
                            docker rm postgres-odoo || true
                        '''
                        // Chạy container PostgreSQL cho Odoo
                        sh '''
                            docker run -d --name postgres-odoo \
                            -e POSTGRES_DB=odoo \
                            -e POSTGRES_USER=odoo \
                            -e POSTGRES_PASSWORD=odoo \
                            -p 5432:5432 \
                            postgres:13
                        '''
                        // Chạy container Odoo mới
                        sh '''
                            docker run -d --name odoo-website \
                            --link postgres-odoo:postgres \
                            -p 8069:8069 \
                            ${IMAGE_TAG}
                        '''
                        // Kiểm tra container có chạy không
                        sleep 10 // Đợi container khởi động
                        sh 'docker ps --filter "name=odoo-website" --format "{{.Names}}" | grep odoo-website'
                        sh 'docker ps --filter "name=postgres-odoo" --format "{{.Names}}" | grep postgres-odoo'
                        echo "Container odoo-website and postgres-odoo are running"
                        // Kiểm tra log của container Odoo
                        sh 'docker logs odoo-website'
                        // Kiểm tra port 8069
                        sh 'netstat -tuln | grep 8069 || echo "Port 8069 not found, check container logs"'
                        // Kiểm tra kết nối tới Odoo
                        sh 'curl -s -f http://localhost:8069 || echo "Cannot connect to Odoo on port 8069, check logs"'
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
            echo 'Truy cập Odoo tại: http://<jenkins-ip>:8069 (hoặc http://localhost:8069 nếu chạy local)'
            // Có thể thêm thông báo qua Slack, email, v.v.
            // slackSend(channel: '#devops', message: "Build #${env.BUILD_NUMBER} deployed successfully!")
        }
        failure {
            echo '❌ Xây dựng hoặc triển khai thất bại.'
            // Có thể thêm thông báo lỗi
            // slackSend(channel: '#devops', message: "Build #${env.BUILD_NUMBER} failed!")
        }
        always {
            // Dọn dẹp các image không cần thiết để tiết kiệm không gian
            sh "docker image prune -f"
        }
    }
}
