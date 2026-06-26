pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Restore') {
            steps { sh 'dotnet restore' }
        }

        stage('SonarQube Begin') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    dotnet sonarscanner begin \
                      /k:"simple-dotnet-web-app" \
                      /d:sonar.login="$SONAR_AUTH_TOKEN" \
                      /d:sonar.exclusions="**/bin/**,**/obj/**,**/*.html,reports/**"
                    '''
                }
            }
        }

        stage('Build') {
            steps { sh 'dotnet build --no-restore' }
        }

        stage('Test') {
            steps { sh 'dotnet test --no-build --verbosity normal' }
        }

        stage('Docker Build') {
            steps { sh 'docker build -t simple-dotnet-web-app .' }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL .'
            }
        }

        stage('Trivy HTML Report') {
            steps {
                sh '''
                wget https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl -O html.tpl &&
                trivy fs --format template --template "@html.tpl" -o trivy-report.html .
                '''
                archiveArtifacts artifacts: 'trivy-report.html', fingerprint: true
            }
        }


        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image \
                --ignore-unfixed \
                --exit-code 1 \
                --severity CRITICAL \
                simple-dotnet-web-app
                '''
            }
        }


        stage('SonarQube End') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    dotnet sonarscanner end \
                      /d:sonar.login="$SONAR_AUTH_TOKEN"
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Sonar Report Link') {
            steps {
                echo "Sonar report: http://localhost:9000/dashboard?id=simple-dotnet-web-app"
            }
        }
    }
}