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

        stage('Run Unit Tests') {
            steps {
                sh '/usr/local/bin/phpunit tests/'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        /opt/sonar-scanner/bin/sonar-scanner \
                        -Dsonar.projectKey=service-app \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=$SONAR_TOKEN \
                        -Dsonar.plugins.downloadOnlyRequired=true \
                        -Dsonar.exclusions=**/*.go \
                        -Dsonar.language=php
                    '''
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
                sh 'trivy image --exit-code 0 --severity CRITICAL $IMAGE_NAME'
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
            sh 'docker rmi $IMAGE_NAME || true'
            sh 'docker image prune -f'
        }

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}