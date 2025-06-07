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
                sh 'docker build -t nodeimage${env.BUILD_NUMBER} .'
            }
        }
        stage('Deploy to Localhost') {
            steps {
                sh '''
                    docker stop odoo-website || true
                    docker rm odoo-website || true
                    docker run -d --name odoo-website -p 8069:8069 nodeimage${env.BUILD_NUMBER}
                '''
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