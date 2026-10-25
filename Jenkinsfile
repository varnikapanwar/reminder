pipeline {
    agent any

    environment {
        IMAGE = 'varnikapanwar/reminder'
        DEPLOYMENT = 'reminder'
        CONTAINER = 'reminder'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE}:${BUILD_NUMBER} .'
                sh 'docker tag ${IMAGE}:${BUILD_NUMBER} ${IMAGE}:latest'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'reminder-dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE}:${BUILD_NUMBER}
                        docker push ${IMAGE}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl set image deployment/${DEPLOYMENT} ${CONTAINER}=${IMAGE}:${BUILD_NUMBER}
                    kubectl rollout status deployment/${DEPLOYMENT} --timeout=120s
                '''
            }
        }
    }

    post {
        success {
            echo 'Reminder CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'Reminder CI/CD pipeline failed. Check the stage logs.'
        }
    }
}
