pipeline {
    agent any
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
        stage('Test Code') {
            steps {
                sh '''
                    npm install
                    npm test
                '''
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