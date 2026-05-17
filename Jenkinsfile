pipeline {
    agent any

    environment {
        // SonarQube configuration
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_TOKEN = credentials('sonar-token')
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
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=digi-jenkins \
                        -Dsonar.projectName=digi-jenkins \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=**/vendor/**,**/tests/** \
                        -Dsonar.tests=tests \
                        -Dsonar.php.tests.reportPath=results.xml \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                    '''
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
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