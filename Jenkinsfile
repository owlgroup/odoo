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
        stage('Install Node.js and npm') {
            steps {
                // Install Node.js and npm
                // The exact command may vary depending on your system and Jenkins setup
                sh 'curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -'
                sh 'sudo apt-get install -y nodejs'
                
                // Verify installation
                sh 'node --version'
                sh 'npm --version'
            }
        }
        stage('Test Code') {
            steps {
                // Install project dependencies
                sh 'npm install'
                
                // Run tests
                sh 'npm test'
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