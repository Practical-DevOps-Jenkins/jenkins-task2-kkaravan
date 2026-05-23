pipeline {
    agent any

    environment {
        APP_PORT = '9090'
    }

    stages {

        stage('Build') {
            steps {
                echo "Job: ${env.JOB_NAME}"
                sh 'mvn clean package'
            }
        }

        stage('Start Application') {
            steps {
                script {

                    // запускаємо Spring Boot
                    sh "java -jar target/*.jar --server.port=${APP_PORT} > app.log 2>&1 &"

                    // даємо час на старт (простий і стабільний варіант)
                    sleep 20

                    // перевіряємо лог (корисно для дебагу)
                    sh 'echo "=== APP LOG START ==="'
                    sh 'cat app.log || true'
                    sh 'echo "=== APP LOG END ==="'
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
                        error("FAIL: RestIT failed")
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
