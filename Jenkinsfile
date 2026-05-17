pipeline {
    agent any

    environment {
        IMAGE_NAME = "maisoonahmed71/service-app:${BUILD_NUMBER}"
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
                    echo "Checking tools..."
                    which docker || echo "Docker NOT FOUND"
                    which php || echo "PHP NOT FOUND"
                '''
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
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        docker run --rm \
                        -v $PWD:/usr/src \
                        sonarsource/sonar-scanner-cli \
                        -Dsonar.projectKey=service-app \
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
                    docker version || true
                    docker build -t $IMAGE_NAME .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    trivy image --exit-code 0 --severity CRITICAL $IMAGE_NAME || true
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
            echo "Pipeline SUCCESS"
        }

        failure {
            echo "Pipeline FAILED"
        }
    }
}