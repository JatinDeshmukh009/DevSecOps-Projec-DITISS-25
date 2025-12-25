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
        IMAGE_REPO = "flask-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
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

        /* ---------- DOCKER STAGES (ADDED) ---------- */

        stage('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    ansiColor('xterm') {
                        sh '''
                            IMAGE_NAME=$DOCKER_USER/$IMAGE_REPO

                            docker build -t $IMAGE_NAME:$IMAGE_TAG .
                            docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                        '''
                    }
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    ansiColor('xterm') {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        '''
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                ansiColor('xterm') {
                    sh '''
                        docker push $IMAGE_NAME:$IMAGE_TAG
                        
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
            sh 'pkill -f app.py || true'
            cleanWs()
        }
    }
}
