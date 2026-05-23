pipeline {
    agent any

    environment {
        APP_PORT = '9090'
        APP_URL = "http://localhost:${APP_PORT}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building project..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Start Application') {
            steps {
                script {

                    // запускаємо app у background
                    sh """
                        nohup java -jar target/*.jar \
                        --server.port=${APP_PORT} \
                        > app.log 2>&1 &
                        echo \$! > app.pid
                    """

                    // ЧЕКАЄМО ПОКИ APP РЕАЛЬНО ПІДНЯВСЯ
                    timeout(time: 90, unit: 'SECONDS') {
                        waitUntil {
                            sh(script: "curl -sf ${APP_URL}/actuator/health > /dev/null", returnStatus: true) == 0
                        }
                    }

                    echo "Application is UP"
                }
            }
        }

        stage('Integration Tests') {
            steps {
                script {
                    try {
                        sh 'mvn test -Dtest=RestIT'
                    } catch (err) {
                        echo "Tests failed"
                        error("Integration tests failed")
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // cleanup app process
                sh '''
                    if [ -f app.pid ]; then
                        kill $(cat app.pid) || true
                    fi
                '''

                echo "Logs:"
                sh 'cat app.log || true'
            }

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
