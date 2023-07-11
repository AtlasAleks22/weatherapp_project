pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']],
                 extensions: [],
                 userRemoteConfigs: [[credentialsId: 'github_auth_id', url: 'https://github.com/AtlasAleks22/weatherapp_project.git']])
            }
        }

        stage('Update Token in environments.ts') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'jenkins_xrapidApi', variable: 'API_KEY')]) {
                        sh "sed -i \"s/api_key_placeholder/$API_KEY/g\" weatherapp_proj/src/app/environments/environment.ts"
                    }
                }
            }
        }

        stage('Image build') {
            steps {
                script {
                    sh 'docker --version'
                    dir('weatherapp_proj') {
                        sh 'docker build -t weather_app .'
                    }
                }
            }
        }

        stage('MegaLinter') {
            options {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE')
            }
            agent {
                docker {
                    image 'oxsecurity/megalinter:v7'
                    args "-u root -e VALIDATE_ALL_CODEBASE=true -v ${WORKSPACE}:/tmp/lint --entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                sh '/entrypoint.sh'
            }
            post {
                always {
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'mega-linter.log,megalinter-reports/**/*', defaultExcludes: false, followSymlinks: false
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run -d -p 4200:4200 weatherapp'
            }
        }

        stage('Run Smoke Test') {
            steps {
                sh "curl -s https://api.weatherapi.com/v1/current.json?key=${API_KEY}&q=Bucharest"
            }
        }

        stage('Login and Push Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(credentialsId: 'docker_auth',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD')
                    ]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        sh 'docker push bejenarudan/weather_app:v1.0'
                    }
                }
            }
        }
    }
}
