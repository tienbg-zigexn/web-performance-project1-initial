pipeline {
    agent any

    parameters {
        booleanParam(name: 'LOCAL_TARGET', defaultValue: true, description: 'Deploy local')
        booleanParam(name: 'REMOTE_TARGET', defaultValue: true, description: 'Deploy remote')
        booleanParam(name: 'FIREBASE_TARGET', defaultValue: true, description: 'Deploy firebase')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building...'
            }
        }

        stage('Lint/Test') {
            steps {
                echo 'Linting and testing...'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def branches = [:]
                    if (params.FIREBASE_TARGET) {
                        branches['Firebase'] = {
                            withCredentials([file(credentialsId: 'adc-credentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                                echo 'Deploying to firebase...'
                            }
                        }
                    }
                    if (params.REMOTE_TARGET) {
                        branches['Remote'] = {
                            echo 'Deploying to Remote'
                        }
                    }
                    if (params.LOCAL_TARGET) {
                        branches['Local'] = {
                            echo 'Deploying to Local'
                        }
                    }
                    parallel branches
                }
            }
        }
    }

    post {
        success {
            script {
                slackSend(message: "Hello world")
            }
        }
        failure {
            script {
                slackSend(message: "Hello world")
            }
        }
    }
}

// vi: shiftwidth=4 tabstop=4
