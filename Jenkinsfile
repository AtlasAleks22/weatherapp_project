pipeline {
    agent any

    environment {
        GITHUB_CREDENTIALS = credentials('573fb6c8-ca66-478c-ae0c-bc561d8c0be1')
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: env.GITHUB_CREDENTIALS, url: 'https://github.com/AtlasAleks22/weatherapp_project.git'

            }
        }

        stage('Build and Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Update Token in environments.ts') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'jenkins-xrapidApi', variable: 'API_KEY')]) {
                        sh "sed -i 's/XRapidAPIKeyHeaderValue: .*/XRapidAPIKeyHeaderValue: \"\\$${API_KEY}\"/' weatherapp_proj/src/app/environments/environment.ts"
                    }
                }
            }
        }

        stage('Run Smoke Test') {
            steps {
                sh "curl -s https://api.weatherapi.com/v1/current.json?key=${API_KEY}&q=Bucharest"
            }
        }

        stage('Build and Deploy') {
            steps {
                sh 'npm run build'
                sh 'npm run deploy'
            }
        }
    }
}
