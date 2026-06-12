pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                sh 'dotnet restore'
            }
        }

        stage('SonarQube Begin') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'dotnet sonarscanner begin /k:"simple-dotnet-web-app"'
                } 
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build --no-restore'
            }
        }

        stage('Test') {
            steps {
                sh 'dotnet test --no-build --verbosity normal'
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
                waitForQualityGate abortPipeline: true
            }
        }
    }
}
