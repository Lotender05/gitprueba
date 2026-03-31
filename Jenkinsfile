pipeline {
    agent any

    environment {
        DOTNET_CLI_HOME = '/tmp/dotnet_cli'
        SONAR_PROJECT_KEY = 'prueba'
        SONAR_HOST_URL = 'http://localhost:9000'
    }

    tools {
        dotnetsdk 'dotnet-10'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                sh 'dotnet restore prueba.slnx'
            }
        }

        stage('SonarQube Begin') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        dotnet sonarscanner begin \
                            /k:"${SONAR_PROJECT_KEY}" \
                            /d:sonar.host.url="${SONAR_HOST_URL}" \
                            /d:sonar.cs.opencover.reportsPaths="**/coverage.opencover.xml"
                    """
                }
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build prueba.slnx --configuration Release --no-restore'
            }
        }

        stage('SonarQube End') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'dotnet sonarscanner end'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
