pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        DOCKER_HOST = 'tcp://host.docker.internal:2375' // Kết nối với Docker Desktop
    }
    stages {
        stage('Debug Environment') {
            steps {
                sh '''
                    echo "Checking environment..."
                    env | grep -E 'DOCKER|GIT'
                    docker -H tcp://host.docker.internal:2375 info
                    ping -c 4 github.com
                    ping -c 4 hub.docker.com
                '''
            }
        }
        stage('Checkout Github') {
            steps {
                // Thêm credentials nếu repository yêu cầu xác thực
                git branch: '18.0', 
                    credentialsId: 'jen-doc-git', // Thay bằng ID credentials trong Jenkins
                    url: 'https://github.com/owlgroup/odoo.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install wheel
                    pip install -r requirements.txt || echo "Failed to install dependencies, check requirements.txt"
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t nodeimage${env.BUILD_NUMBER} ."
            }
        }
        stage('Deploy to Localhost') {
            steps {
                sh """
                    docker stop odoo-website || true
                    docker rm odoo-website || true
                    docker run -d --name odoo-website -p 8069:8069 nodeimage${env.BUILD_NUMBER}
                """
            }
        }
    }
    post {
        success {
            echo '✅ Build and deployment success!'
        }
        failure {
            echo '❌ Build or deployment failed. Check logs.'
        }
    }
}