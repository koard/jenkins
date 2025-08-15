pipeline {
  agent {
    docker {
      image 'python:3.11'
      args  '-u 0:0'          // รัน container เป็น root เพื่อเลี่ยงสิทธิ์ไฟล์บน workspace
    }
  }
  options { timestamps() }
  environment {
    PIP_CACHE_DIR = "${WORKSPACE}/.pip-cache"
    HOME = "${WORKSPACE}"     // กัน pip ไปเขียน /.local
  }
  stages {
    stage('Checkout') {
      steps {
        // เปลี่ยน URL/branch ตามรีโปของคุณ
        git branch: 'main', url: 'https://github.com/koard/jenkins.git'
      }
    }

    stage('Prep workspace perms (1 ครั้งต่อรัน)') {
      steps {
        sh '''
          # ทำให้โฟลเดอร์ทำงานเขียนได้ (กัน edge cases เรื่อง permission)
          chmod -R a+rwX .
        '''
      }
    }

    stage('Setup venv & Install deps') {
      steps {
        sh '''
          python -m venv .venv
          . .venv/bin/activate
          pip install --upgrade pip
          # ถ้ามี requirements.txt ใช้บรรทัดนี้แทน:
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
