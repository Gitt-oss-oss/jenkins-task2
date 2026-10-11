pipeline {
    agent any

    environment {
        APP_PORT = '9090'
        APP_DIR  = "${env.JOB_NAME}"
    }

    stages {
        stage('Build') {
            steps {
                echo "START Build Jobname=${env.JOB_NAME}"
                sh 'mvn -B package -DskipTests'
                stash includes: 'target/*.war', name: 'war_file'
            }
        }

        stage('Integration Test') {
            parallel {
                stage('Running Application') {
                    agent any
                    steps {
                        echo "START Application"
                        script {
                            try {
                                timeout(time: 60, unit: 'SECONDS') {
                                    unstash 'war_file'
                                    dir("target") {
                                        sh 'pwd'
                                        sh "SERVER_PORT=${env.APP_PORT} java -jar contact.war"
                                    }
                                }
                            } catch (Throwable e) {
                                echo "Application stage stopped: ${e.getMessage()}"
                                currentBuild.result = 'SUCCESS'
                            }
                        }
                    }
                }
                stage('Running Test') {
                    steps {
                        sleep time: 30, unit: 'SECONDS'
                        echo "START Test"
                        sh 'mvn -B test -Dtest=RestIT'
                        echo "Done Test"
                    }
                }
            }
        }
    }
}

