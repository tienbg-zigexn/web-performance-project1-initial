pipeline {
    agent any

    parameters {
        booleanParam(name: 'LOCAL_TARGET', defaultValue: true, description: 'Deploy local')
        booleanParam(name: 'REMOTE_TARGET', defaultValue: true, description: 'Deploy remote')
        booleanParam(name: 'FIREBASE_TARGET', defaultValue: true, description: 'Deploy firebase')
    }

    environment {
        SLACK_CREDENTIAL = credientials('jenkins-slack-token')
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
                    def deployed = []
                    if (params.FIREBASE_TARGET) {
                        branches['Firebase'] = {
                            withCredentials([file(credentialsId: 'adc-credentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                                echo 'Deploying to firebase...'
                            }
                        }
                        deployed.add('firebase')
                    }
                    if (params.REMOTE_TARGET) {
                        branches['Remote'] = {
                            echo 'Deploying to Remote'
                        }
                        deployed.add('remote')
                    }
                    if (params.LOCAL_TARGET) {
                        branches['Local'] = {
                            echo 'Deploying to Local'
                        }
                        deployed.add('local')
                    }
                    parallel branches
                    env.DEPLOYED_TARGETS = deployed.join(',')
                }
            }
        }
    }

    post {
        success {
            script {
                def blocks = [
                    [
                        'type': 'section',
                        'text': [
                            'type': 'mrkdwn',
                            'text': "*Build successful* by TienBG. Jenkins: ${env.BUILD_URL}"
                        ]
                    ]
                ]
                def targets = env.DEPLOYED_TARGETS?.split(',') ?: []
                targets.each { target ->
                    def url = target == 'firebase' ? 'https://tienbg-workshop2.firebaseapp.com' :
                              target == 'remote' ? 'https://remote.example.com' :
                              target == 'local' ? 'http://localhost:3000' : 'unknown'
                    blocks.add([
                        'type': 'section',
                        'text': [
                            'type': 'mrkdwn',
                            'text': "*${target.capitalize()}*: ${url}"
                        ]
                    ])
                }
                slackSend(tokenCredentialId: "$SLACK_CREDENTIAL", blocks: blocks)
            }
        }
        failure {
            script {
                def message = "*Build failed* by TienBG. Jenkins: ${env.BUILD_URL}"
                slackSend(tokenCredentialId: "$SLACK_CREDENTIAL", blocks: [
                    [
                        'type': 'section',
                        'text': [
                            'type': 'mrkdwn',
                            'text': message
                        ]
                    ]
                ])
            }
        }
    }
}

// vi: shiftwidth=4 tabstop=4
