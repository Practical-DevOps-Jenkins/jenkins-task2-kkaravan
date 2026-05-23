pipeline {
    agent any

    environment {
        APP_PORT = '9090'
    }

    stages {

        stage('Build') {
            steps {
                echo "Building project..."
                echo "Job name: ${env.JOB_NAME}"
                sh 'mvn clean package'
            }
        }

        stage('Start Application') {
            steps {
                script {
                    // запускаємо застосунок у background
                    sh "java -jar target/*.jar --server.port=${APP_PORT} > app.log 2>&1 &"

                    // чекаємо поки порт стане доступний
                    timeout(time: 60, unit: 'SECONDS') {
                        waitUntil {
                            sh(script: "nc -z localhost ${APP_PORT}", returnStatus: true) == 0
                        }
                    }

                    echo "Application is up on port ${APP_PORT}"
                }
            }
        }

        stage('Integration Test (RestIT)') {
            steps {
                script {
                    try {
                        timeout(time: 30, unit: 'SECONDS') {
                            sh 'mvn test -Dtest=RestIT'
                        }
                    } catch (err) {
                        echo "Integration tests failed or timed out: ${err}"
                        error("Tests failed")
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished for job: ${env.JOB_NAME}"
        }
        success {
            echo "BUILD SUCCESS"
        }
        failure {
            echo "BUILD FAILURE"
        }
    }
}
