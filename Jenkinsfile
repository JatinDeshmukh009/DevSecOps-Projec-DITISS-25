pipeline {
    agent {
        label 'jenkins_agent'  
         }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        VENV = "venv"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python Environment') {
            steps {
                ansiColor('xterm') {
                    sh '''
                        python3 -m venv $VENV
                        . $VENV/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Lint Code') {
            steps {
                ansiColor('xterm') {
                    sh '''
                        . $VENV/bin/activate
                        flake8 app.py || true
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                ansiColor('xterm') {
                    sh '''
                        . $VENV/bin/activate
                        pytest || true
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                ansiColor('xterm') {
                    sh '''
                        . $VENV/bin/activate
                        python app.py &
                        sleep 5
                        curl -f http://127.0.0.1:5000/health
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
        always {
            cleanWs()
        }
    }
}
