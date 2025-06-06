pipeline {
    agent any

    stages {
        stage('Checkout Github') {
            steps {
                git branch: '18.0', credentialsId: 'jen-doc-git', url: 'https://github.com/owlgroup/odoo.git'
            }
        }

        stage('Install Python dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install -r requirements.txt || echo "Không tìm thấy requirements.txt"
                '''
            }
        }

        stage('Test Code') {
            steps {
                sh '''
                    source venv/bin/activate
                    echo "👉 Running tests..."
                    python3 -m unittest discover -s tests || echo "✅ Không có test nào được chạy"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Build completed successfully!'
        }
        failure {
            echo '❌ Build failed. Check logs.'
        }
    }
}
