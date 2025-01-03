pipeline {
    agent any

    environment {
        PYTHON_VERSION = '3.9'
    }

    parameters {
        string(name: 'THRESHOLD', defaultValue: '85', description: 'Disk usage threshold')
        string(name: 'LOG_RETENTION_DAYS', defaultValue: '30', description: 'Log retention period (days)')
        string(name: 'SCAN_PATH', defaultValue: '/', description: 'Directory to scan')
        booleanParam(name: 'CHECK_ZOMBIES', defaultValue: true, description: 'Enable zombie process detection')
    }

    stages {
        stage('Setup Python Environment') {
            steps {
                sh 'python3 -m venv venv'
                sh './venv/bin/pip install -r requirements.txt'
            }
        }
        stage('Run Monitoring Script') {
            steps {
                sh '''
                source venv/bin/activate
                python main.py --threshold ${THRESHOLD} \
                               --log_retention_days ${LOG_RETENTION_DAYS} \
                               --scan_path ${SCAN_PATH} \
                               --check_zombies ${CHECK_ZOMBIES}
                '''
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
