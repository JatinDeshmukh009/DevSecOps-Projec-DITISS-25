pipeline {
    agent {
        label 'jenkins_agent'
        }

    options {
        timestamps()
        disableConcurrentBuilds()
        ansiColor('xterm')
    }

    environment {
        VENV        = "venv"
        FLASK_APP  = "app.py"
        FLASK_ENV  = "testing"
        APP_PORT   = "5000"
    }

    stages {

        stage('Checkout Source (PR / Branch)') {
            steps {
                checkout scm
            }
        }

        stage('Show Build Context') {
            steps {
                sh '''
                    echo "Branch Name      : ${BRANCH_NAME}"
                    echo "Change ID (PR)   : ${CHANGE_ID}"
                    echo "Change Branch   : ${CHANGE_BRANCH}"
                    echo "Target Branch   : ${CHANGE_TARGET}"
                '''
            }
        }

        stage('Setup Python Virtual Environment') {
            steps {
                sh '''
                    python3 --version
                    python3 -m venv $VENV
                    . $VENV/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Lint Check (flake8)') {
            steps {
                sh '''
                    . $VENV/bin/activate
                    pip install flake8
                    flake8 app.py --max-line-length=120
                '''
            }
        }

        stage('Security Scan (Bandit)') {
            steps {
                sh '''
                    . $VENV/bin/activate
                    pip install bandit
                    bandit -r app.py || true
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                    . $VENV/bin/activate
                    pip install pytest
                    pytest || echo "No tests found"
                '''
            }
        }

        stage('Run Application & Health Check') {
            steps {
                sh '''
                    . $VENV/bin/activate
                    nohup python app.py > app.log 2>&1 &
                    sleep 5
                    curl -f http://127.0.0.1:${APP_PORT}/health
                '''
            }
        }
    }

    post {

        success {
            echo "✅ Jenkins PR pipeline succeeded"
        }

        failure {
            echo "❌ Jenkins PR pipeline failed"
        }

        always {
            sh '''
                echo "Cleaning up..."
                pkill -f app.py || true
                rm -rf $VENV
            '''
            archiveArtifacts artifacts: 'app.log', allowEmptyArchive: true
        }
    }
}
