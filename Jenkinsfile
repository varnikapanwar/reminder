pipeline {
    agent any

    environment {
        IMAGE = 'varnikapanwar/reminder'
        DOCKER_CREDENTIALS = 'reminder-dockerhub-creds'
    }

    stages {
        stage('Validate Branch') {
            steps {
                script {
                    if (!(env.BRANCH_NAME in ['v1', 'v2', 'v3'])) {
                        error("Unsupported branch: ${env.BRANCH_NAME}")
                    }
                    env.VERSION = env.BRANCH_NAME
                    env.DEPLOYMENT = "reminder-${env.VERSION}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE}:${VERSION} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE}:${VERSION}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl set image deployment/${DEPLOYMENT} reminder=${IMAGE}:${VERSION}
                    kubectl rollout status deployment/${DEPLOYMENT} --timeout=120s
                '''
            }
        }
    }

    post {
        success {
            echo "Reminder ${VERSION} pipeline completed successfully."
        }
        failure {
            echo 'Reminder pipeline failed. Check the stage logs.'
        }
    }
}
