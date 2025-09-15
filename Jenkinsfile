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
                            sh '''
                                /usr/bin/mkdir -p public
                                cp -rf index.html 404.html css js images public/
                            '''
                            withCredentials([file(credentialsId: 'adc-credentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                                sh 'firebase deploy --only hosting --project=tienbg-workshop2'
                            }
                        }
                        deployed.add('firebase')
                    }
                    if (params.REMOTE_TARGET) {
                        branches['Remote'] = {
                            ansiblePlaybook(
                                playbook: '/var/jenkins_home/ansible/deploy_workshop2.yml',
                                inventory: '/var/jenkins_home/ansible/hosts',
                                extraVars: [
                                    TARGET_SERVERS: 'direct'
                                ]
                            )
                        }
                        deployed.add('remote')
                    }
                    if (params.LOCAL_TARGET) {
                        branches['Local'] = {
                            ansiblePlaybook(
                                playbook: '/var/jenkins_home/ansible/deploy_workshop2.yml',
                                inventory: '/var/jenkins_home/ansible/hosts',
                                extraVars: [
                                    TARGET_SERVERS: 'local'
                                ]
                            )
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
                              target == 'remote' ? 'http://10.1.1.195/jenkins/tienbg2/current' :
                              target == 'local' ? 'http://localhost' : 'unknown'
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
