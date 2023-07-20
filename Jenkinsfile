pipeline {
    agent any

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
                    args "-u root -e VALIDATE_ALL_CODEBASE=true -v ${WORKSPACE}:/tmp/lint -v ${WORKSPACE}/weatherapp_proj:/tmp/lint/weatherapp_proj --entrypoint=''"
                    reuseNode true
                    customWorkspace '/tmp/lint'
                }
            }
            steps {
                sh '/entrypoint.sh'
            }
            post {
                always {
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'mega-linter.log', defaultExcludes: false, followSymlinks: false
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'megalinter-reports/**/*.html', defaultExcludes: false, followSymlinks: false
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run -d -p 4200:4200 weather_app'
            }
        }

        stage('Run Smoke Test') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'jenkins_xrapidApi', variable: 'API_KEY')]) {
                        def weatherApiBaseUrl = 'https://openweather43.p.rapidapi.com/weather'
                        def XRapidAPIHostHeaderValue = 'openweather43.p.rapidapi.com'
                        sh "curl -s -H 'X-RapidAPI-Host: ${XRapidAPIHostHeaderValue}' -H 'X-RapidAPI-Key: ${API_KEY}' '${weatherApiBaseUrl}?q=Bucharest'"
                        sh "curl -s -H 'X-RapidAPI-Host: ${XRapidAPIHostHeaderValue}' -H 'X-RapidAPI-Key: ${API_KEY}' '${weatherApiBaseUrl}?q=Timisoara'"
                    }
                }
            }
        }

        stage('Login and Push Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(credentialsId: 'docker_auth',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD')
                    ])
                    {
                        sh "docker login -u ${DOCKER_USERNAME} -p \$DOCKER_PASSWORD"
                        sh "docker tag weather_app ${DOCKER_USERNAME}/weather_app:v1.0"
                        sh "docker push ${DOCKER_USERNAME}/weather_app:v1.0"
                    }
                }
            }
        }
        stage('Deploy with Docker Compose') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'jenkins_xrapidApi', variable: 'API_KEY')]) {
                        withEnv(['XRapidAPIKeyHeaderValue=$API_KEY']) {
                            sh 'docker-compose -f ./weatherapp_proj/docker-compose.yml up -d'
                        }
                    }
                    echo 'Application deployed successfully!'
                    echo 'Access the application at: http://localhost:4200'
                }
            }
        }
}
