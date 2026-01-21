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
        IMAGE_TAG  = "latest"
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
                        script {
                            env.IMAGE_NAME = "${DOCKER_USER}/${IMAGE_REPO}"
                        }
                        
                        sh '''
                            docker build -t $IMAGE_NAME:$IMAGE_TAG .
                            docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                        '''
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh '''
                echo "======================================"
                echo " Trivy Image Security Scan"
                echo " Image: $IMAGE_NAME:$IMAGE_TAG"
                echo "======================================"

                trivy image \
                    --severity HIGH,CRITICAL \
                    --format template \
                    --template "@contrib/html.tpl" \
                    --output trivy-report.html \
                    --no-progress \
                    $IMAGE_NAME:$IMAGE_TAG
                '''
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
        always {
            archiveArtifacts artifacts: 'trivy-report.html', fingerprint: true
            sh 'pkill -f app.py || true'
            cleanWs()
        }
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
