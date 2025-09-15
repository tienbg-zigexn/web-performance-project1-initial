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
                def branch = env.BRANCH_NAME ?: 'unknown'
                def duration = currentBuild.durationString
                def cause = currentBuild.getBuildCauses()[0]?.shortDescription ?: 'unknown'
                def details = ":scout-approved: *Build Successful*\n\n*Branch:* ${branch}\n*Duration:* ${duration}\n*Cause:* ${cause}\n\n*Deployed Targets:*\n"
                def targets = env.DEPLOYED_TARGETS?.split(',') ?: []
                targets.each { target ->
                    def url = target == 'firebase' ? 'https://tienbg-workshop2.firebaseapp.com' :
                              target == 'remote' ? 'https://remote.example.com' :
                              target == 'local' ? 'http://localhost:3000' : 'unknown'
                    details += "• *${target.capitalize()}:* <${url}|${url}>\n"
                }
                slackSend(tokenCredentialId: 'jenkins-slack-token', message: "Build Successful", attachments: [[color: "good", text: details.trim()]])
            }
        }
        failure {
            script {
                def branch = env.BRANCH_NAME ?: 'unknown'
                def duration = currentBuild.durationString
                def cause = currentBuild.getBuildCauses()[0]?.shortDescription ?: 'unknown'
                def details = ":evil-grin: *Build Failed*\n\n*Branch:* ${branch}\n*Duration:* ${duration}\n*Cause:* ${cause}\n\n*Jenkins:* <${env.BUILD_URL}|View Build>"
                slackSend(tokenCredentialId: 'jenkins-slack-token', message: "Build Failed", attachments: [[color: "danger", text: details]])
            }
        }
    }
}

// vi: shiftwidth=4 tabstop=4
