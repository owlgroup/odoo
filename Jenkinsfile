pipeline {
    agent any
    environment {
        DOCKER_HOST = 'tcp://khoadue.me:2375'  // Thay thế bằng URL Docker daemon của bạn
    }
    triggers {
        githubPush()
    }
    stages {
        stage('Checkout Github') {
            steps {
                git branch: '18.0', url: 'https://github.com/owlgroup/odoo.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install wheel
                    pip install -r requirements.txt
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("nodeimage${env.BUILD_NUMBER}")
                }
            }
        }
    }
    post {
        success {
            echo '✅ Build success!'
        }
        failure {
            echo '❌ Build failed. Check logs.'
        }
    }
}