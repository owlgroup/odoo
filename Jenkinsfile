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
                // Install Node.js and npm using nvm
                // This assumes nvm is already installed on the Jenkins agent
                sh '''
                    export NVM_DIR="$HOME/.nvm"
                    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
                    nvm install 14
                    nvm use 14
                '''
                
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