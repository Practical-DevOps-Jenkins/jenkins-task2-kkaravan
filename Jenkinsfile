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

        stage('Run Application') {
            steps {
                script {
                    try {
                        timeout(time: 60, unit: 'SECONDS') {
                            dir('target') {
                                sh "java -jar *.jar --server.port=${APP_PORT}"
                            }
                        }
                    } catch (err) {
                        echo "Application stopped or timed out: ${err}"
                    }
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
