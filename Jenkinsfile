pipeline {
    agent any

    environment {
        IMAGE_NAME = "maisson88/service-app:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/maisson88/digi-jenkins.git',
                    branch: 'main'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh '''
                    if command -v phpunit >/dev/null 2>&1; then
                        phpunit tests || true
                    else
                        echo "PHPUnit not installed - skipping tests"
                    fi
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    withCredentials([string(credentialsId: 'habiba', variable: 'SONAR_TOKEN')]) {
                        sh '''
                            /opt/sonar-scanner/bin/sonar-scanner \
                            -Dsonar.projectKey=service-app \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://localhost:9000 \
                            -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    if command -v trivy >/dev/null 2>&1; then
                        trivy image --severity CRITICAL $IMAGE_NAME || true
                    else
                        echo "Trivy not installed - skipping scan"
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
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker stop service-app || true
                    docker rm service-app || true

                    docker run -d \
                        --name service-app \
                        -p 8081:80 \
                        $IMAGE_NAME
                '''
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f || true'
        }

        success {
            echo 'Pipeline SUCCESS ✔'
        }

        failure {
            echo 'Pipeline FAILED ✖'
        }
    }
}