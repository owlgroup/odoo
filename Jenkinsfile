pipeline {
    agent any
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
                sh 'docker build -t khoadue.me/odoo-website:latest .'
            }
        }
        stage('Deploy to Docker') {
            steps {
                sh 'docker stop khoadue.me/odoo-website || true'
                sh 'docker rm khoadue.me/odoo-website || true'
                sh 'docker run -d --name khoadue.me/odoo-website -p 8069:8069 khoadue.me/odoo-website:latest'
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