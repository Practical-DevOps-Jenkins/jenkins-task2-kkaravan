pipeline {
    agent any

    environment {
        APPPORT = '9090'
    }

    stages {
        stage('Build') {
            steps {
                echo "Building project..."
                echo "Job name: ${env.JOBNAME}"
                sh 'mvn clean package'
            }
        }

        stage('Check Logs for FAILURE') {
            steps {
                script {
                    def logFile = "/local/jenkinsHome/jobs/job/builds/1/log"
                    def failureDetected = false
                    try {
                        sh "grep -q \"FAILURE\" ${logFile}"
                        failureDetected = true
                    } catch (err) {
                        failureDetected = false
                    }
                    if (failureDetected) {
                        echo "WARNING: 'FAILURE' found in log file."
                    } else {
                        echo "No 'FAILURE' found in log file."
                    }
                }
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
                                        sh "java -jar *.jar --server.port=${APPPORT}"
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
            echo "Pipeline finished for job: ${env.JOBNAME}"
        }
    }
}
