pipeline {
    agent any

    parameters {
        booleanParam(name: 'LOCAL_TARGET', defaultValue: true, description: 'Deploy local')
        booleanParam(name: 'REMOTE_TARGET', defaultValue: true, description: 'Deploy remote')
        booleanParam(name: 'FIREBASE_TARGET', defaultValue: true, description: 'Deploy firebase')
    }

    triggers { pollSCM('* * * * *') }

    stages {
        stage('Checkout') {
            steps {
                script {
                    def scmVars = checkout scm
                    env.BRANCH_NAME = scmVars.GIT_BRANCH
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Lint/Test') {
            steps {
                sh 'npm run test:ci'
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
                                sh 'firebase deploy --only hosting --project=tienbg-workshop2'
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
                slackSend(message: details.trim(), color: 'good')
            }
        }
        failure {
            script {
                def branch = env.BRANCH_NAME ?: 'unknown'
                def duration = currentBuild.durationString
                def cause = currentBuild.getBuildCauses()[0]?.shortDescription ?: 'unknown'
                def details = ":evil-grin: *Build Failed*\n\n*Branch:* ${branch}\n*Duration:* ${duration}\n*Cause:* ${cause}\n\n*Jenkins:* <${env.BUILD_URL}|View Build>"
                slackSend(message: details.trim(), color: 'danger')
            }
        }
    }
}

// vi: shiftwidth=4 tabstop=4
