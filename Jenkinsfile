pipeline {
    agent any

    environment {
        GITHUB_CREDENTIALS = credentials('573fb6c8-ca66-478c-ae0c-bc561d8c0be1')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']],
                 extensions: [], 
                 userRemoteConfigs: [[credentialsId: 'github_auth_id', url: 'https://github.com/AtlasAleks22/weatherapp_project.git']])

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
                        sh "sed -i \"s/api_key_placeholder/$jenkins_xrapidApi/g\" weatherapp_proj/src/app/environments/environment.ts"
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
