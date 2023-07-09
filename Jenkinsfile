pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/AtlasAleks22/weatherapp_project.git'
            }
        }

        stage('Build and Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Update Token in environments.ts') {
            steps {
                withCredentials([string(credentialsId: 'jenkins_xrapidApi', variable: 'API_KEY')]) {
                    sh "sed -i 's/XRapidAPIKeyHeaderValue: .*\$/XRapidAPIKeyHeaderValue: \"\${API_KEY}\"/' weatherapp_proj/src/app/environments/environment.ts"
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
