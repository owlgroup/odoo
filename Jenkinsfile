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
                    echo "Checking environment..."
                    unset DOCKER_TLS_VERIFY
                    unset DOCKER_CERT_PATH
                    export DOCKER_TLS_VERIFY=0
                    env | grep -E 'DOCKER|GIT'
                    curl -s http://host.docker.internal:2375/_ping || echo "Failed to ping Docker daemon via HTTP"
                    DOCKER_HOST=tcp://host.docker.internal:2375 docker info || echo "Failed to connect to Docker daemon"
                    curl -Is https://github.com | head -n 1
                    curl -Is https://hub.docker.com | head -n 1
                '''
            }
        }
        stage('Checkout Github') {
            steps {
                git branch: '18.0', credentialsId: 'jen-doc-git', url: 'https://github.com/owlgroup/odoo.git'
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
                sh '''
                    unset DOCKER_TLS_VERIFY
                    unset DOCKER_CERT_PATH
                    export DOCKER_TLS_VERIFY=0
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
                    DOCKER_HOST=tcp://host.docker.internal:2375 docker stop odoo-website || true
                    DOCKER_HOST=tcp://host.docker.internal:2375 docker rm odoo-website || true
                    DOCKER_HOST=tcp://host.docker.internal:2375 docker run -d --name odoo-website -p 8069:8069 "$IMAGE_TAG"
                '''
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