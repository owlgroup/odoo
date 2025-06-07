pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        DOCKER_HOST = 'tcp://host.docker.internal:2375'
        DOCKER_TLS_VERIFY = ''
        DOCKER_CERT_PATH = ''
    }
    stages {
        stage('Debug Environment') {
            steps {
                sh '''
                    echo "Checking environment..."
                    unset DOCKER_TLS_VERIFY
                    unset DOCKER_CERT_PATH
                    env | grep -E 'DOCKER|GIT'
                    docker info
                    curl -Is https://github.com | head -n 1
                    curl -Is https://hub.docker.com | head -n 1
                '''
            }
        }
        stage('Checkout Github') {
            steps {
                git branch: '18.0', credentialsId: 'github-credentials-id', url: 'https://github.com/owlgroup/odoo.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip wheel
                    pip install -r requirements.txt || echo "Failed to install dependencies"
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
            echo '❌ Build or deployment failed.'
        }
    }
}