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
        stage('Prepare Addons') {
            steps {
                sh '''
                    . venv/bin/activate
                    # Kiểm tra và tạo thư mục custom-addons nếu chưa tồn tại
                    if [ ! -d "custom-addons" ]; then
                        echo "Thư mục custom-addons không tồn tại. Tạo thư mục mới."
                        mkdir custom-addons
                    fi
                    # Sao chép addons từ repository vào thư mục custom-addons nếu cần
                    # Ví dụ:
                    # cp -r /path/to/your/addons/* custom-addons/
                '''
            }
        }
        stage('Test Code') {
            steps {
                sh '''
                    . venv/bin/activate
                    # Kiểm tra xem thư mục custom-addons có chứa các addons hợp lệ hay không
                    if [ -z "$(ls -A custom-addons)" ]; then
                        echo "Thư mục custom-addons không chứa addons hợp lệ. Vui lòng thêm addons vào thư mục này."
                        exit 1
                    fi
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