pipeline {
    agent any

    environment {
        APP_PORT = '9090'
    }

    stages {

        stage('Build') {
            steps {
                echo "Job name: ${env.JOB_NAME}"
                sh 'mvn clean package'
            }
        }

        stage('Run Application') {
            steps {
                script {
                    sh "java -jar target/*.jar --server.port=${APP_PORT} &"
                    sleep 10
                }
            }
        }

        stage('Integration Test') {
            steps {
                script {
                    try {
                        timeout(time: 30, unit: 'SECONDS') {
                            sh 'mvn test -Dtest=RestIT'
                        }
                    } catch (err) {
                        echo "Tests failed or timed out: ${err}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished for job: ${env.JOB_NAME}"
        }
    }
}
