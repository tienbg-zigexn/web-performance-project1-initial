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
                blocks = [
                    [
                        'type': 'section',
                        'text': [
                            'type': 'mrkdwn',
                            'text': "Hello, Assistant to the Regional Manager Dwight! *Michael Scott* wants to know where you'd like to take the Paper Company investors to dinner tonight.\n\n *Please select a restaurant:*"
                        ]
                    ],
                    [
                        'type': 'divider'
                    ],
                    [
                        'type': 'section',
                        'text': [
                            'type': 'mrkdwn',
                            'text': "*Farmhouse Thai Cuisine*\n:star::star::star::star: 1528 reviews\n They do have some vegan options, like the roti and curry, plus they have a ton of salad stuff and noodles can be ordered without meat!! They have something for everyone here"
                        ],
                        'accessory': [
                            'type': 'image',
                            'image_url': 'https://s3-media3.fl.yelpcdn.com/bphoto/c7ed05m9lC2EmA3Aruue7A/o.jpg',
                            'alt_text': 'alt text for image'
                        ]
                    ]
                ]

                slackSend(blocks: blocks)
            }
        }
        failure {
            script {
                slackSend(message: 'Hello world')
            }
        }
    }
}

// vi: shiftwidth=4 tabstop=4
