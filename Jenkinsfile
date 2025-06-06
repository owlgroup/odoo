pipeline {
    agent any

    stages {
        stage('Checkout Github') {
            steps {
                git branch: '18.0', url: 'https://github.com/owlgroup/odoo.git'
            }
        }

        stage('Install Python dependencies') {
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
                    . venv/bin/activate
                    python3 odoo/odoo-bin --test-enable --stop-after-init -d test_db --addons-path=addons
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
