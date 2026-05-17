pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://host.docker.internal:9000'  // Use this for Docker container
        // OR use your host IP: 'http://192.168.x.x:9000'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/maisson88/digi-jenkins.git',
                    credentialsId: 'github-pat-creds'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run sonar-scanner using Docker
                    sh '''
                        docker run --rm \
                            -v ${PWD}:/usr/src \
                            sonarsource/sonar-scanner-cli \
                            -Dsonar.projectKey=digi-jenkins \
                            -Dsonar.projectName=digi-jenkins \
                            -Dsonar.sources=/usr/src \
                            -Dsonar.host.url=${SONAR_HOST_URL}
                    '''
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh '/usr/local/bin/phpunit --log-junit results.xml tests/'
                }
            }
        }

        stage('Display Results') {
            steps {
                junit 'results.xml'
            }
        }
    }

    post {
        always {
            echo 'Pipeline job finished.'
        }
        success {
            echo 'Congratulations! All tests passed successfully.'
        }
        failure {
            echo 'The code failed the tests! Please check the Test Result trend for details.'
        }
    }
}