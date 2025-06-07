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
                    # Đảm bảo thư mục custom-addons tồn tại trong workspace
                    if [ ! -d "custom-addons" ]; then
                        echo "Thư mục custom-addons không tồn tại. Tạo thư mục mới."
                        mkdir custom-addons
                    fi
                    # Chạy Odoo với thư mục addons và custom-addons
                    python3 odoo-bin --test-enable --stop-after-init -d test_db --addons-path=addons,custom-addons
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