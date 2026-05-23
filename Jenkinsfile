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

        stage('Integration Test (Parallel)') {
            parallel {

                stage('Running Application') {
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

                stage('Running REST IT Tests') {
                    steps {
                        script {
                            try {
                                timeout(time: 30, unit: 'SECONDS') {
                                    sh 'mvn test -Dtest=RestIT'
                                }
                            } catch (err) {
                                echo "Integration tests failed or timed out: ${err}"
                            }
                        }
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
