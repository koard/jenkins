pipeline {
    agent { docker { image 'python:3.11' } }
    options { timestamps() }
    environment {
        PIP_CACHE_DIR = "${WORKSPACE}/.pip-cache"
        HOME = "${WORKSPACE}"   // ป้องกัน pip ไปเขียนที่ /.local
    }
    stages {
        stage('Checkout') {
            steps {
                // ====== กรณี PUBLIC repo ======
                git branch: 'main', url: 'https://github.com/koard/jenkins.git'

                // ====== ถ้าเป็น PRIVATE ให้ใช้แบบนี้แทน ======
                // git branch: 'main',
                //     url: 'https://github.com/<OWNER>/<REPO>.git',
                //     credentialsId: 'MY_GIT_CREDENTIALS'
            }
        }

        stage('Setup venv & Install deps') {
            steps {
                sh '''
                    python -m venv .venv
                    . .venv/bin/activate
                    pip install --upgrade pip
                    # ถ้ารีโปมี requirements.txt ให้ใช้บรรทัดนี้แทน:
                    # pip install -r requirements.txt
                    pip install -U pytest pytest-html openpyxl pandas
                '''
            }
        }

        stage('Run tests') {
            steps {
                sh '''
                    . .venv/bin/activate
                    mkdir -p reports
                    pytest -q \
                        --junitxml=reports/junit.xml \
                        --html=reports/report.html --self-contained-html
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'reports/*', fingerprint: true
            junit 'reports/junit.xml'
        }
    }
}