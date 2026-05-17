pipeline {
    agent any

    environment {
        IMAGE_NAME = "maisoonahmed71/service-app:${BUILD_NUMBER}"
        SONAR_SCANNER_HOME = "/opt/sonar-scanner"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-pat-creds',
                    url: 'https://github.com/maisson88/digi-jenkins.git'
            }
        }

        stage('Debug Tools') {
            steps {
                sh '''
                    echo "PATH=$PATH"
                    which docker || echo "Docker NOT FOUND"
                    which php || echo "PHP NOT FOUND"
                    which sonar-scanner || echo "Sonar NOT FOUND"
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        export PATH=$PATH:/opt/sonar-scanner/bin

                        /opt/sonar-scanner/bin/sonar-scanner \
                            -Dsonar.projectKey=service-app \
                            -Dsonar.projectName=service-app \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://localhost:9000 \
                            -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    if command -v docker >/dev/null 2>&1; then
                        docker build -t $IMAGE_NAME .
                    else
                        echo "Docker not installed - skipping build"
                    fi
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    if command -v trivy >/dev/null 2>&1; then
                        trivy image --exit-code 0 --severity CRITICAL $IMAGE_NAME || true
                    else
                        echo "Trivy not installed"
                    fi
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        if command -v docker >/dev/null 2>&1; then
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                            docker push $IMAGE_NAME
                        else
                            echo "Docker not available - skipping push"
                        fi
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    if command -v docker >/dev/null 2>&1; then
                        docker stop service-app || true
                        docker rm service-app || true

                        docker run -d \
                            --name service-app \
                            -p 8081:80 \
                            $IMAGE_NAME
                    else
                        echo "Docker not available - skipping deploy"
                    fi
                '''
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f || true'
        }

        success {
            echo "Pipeline SUCCESS"
        }

        failure {
            echo "Pipeline FAILED"
        }
    }
}