pipeline {

    agent {
        label 'ec2-agent2'
    }

    environment {
        IMAGE_NAME = "maisoonahmed71/service-app:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-pat-creds',
                    url: 'https://github.com/maisson88/digi-jenkins.git'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'phpunit tests || true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=service-app \
                        -Dsonar.sources=src
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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
                sh 'trivy image --exit-code 1 --severity CRITICAL $IMAGE_NAME'
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
                        -p 8081:8000 \
                        $IMAGE_NAME
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rmi $IMAGE_NAME || true'
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