pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

    
        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL .'
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
                    sh 'dotnet sonarscanner begin /k:"simple-dotnet-web-app" /d:sonar.exclusions="**/bin/**,**/obj/**,**/*.html,reports/**"' 
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

        

        stage('Trivy HTML Report') {
            steps {
                sh '''
                wget https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl -O html.tpl &&
        
                trivy fs \
                --format template \
                --template "@html.tpl" \
                -o trivy-report.html \
                .
                '''
                archiveArtifacts artifacts: 'trivy-report.html', fingerprint: true
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


        stage('Docker Build') {
            steps {
                sh 'docker build -t simple-dotnet-web-app .'
            }
        }


        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL simple-dotnet-web-app'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
            }
        }


        stage('Sonar Report Link') {
            steps {
                echo "Sonar report: http://localhost:9000/dashboard?id=simple-dotnet-web-app"
            }
        }

}
