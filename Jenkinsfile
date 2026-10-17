pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'mrunal2804'
        BACKEND_IMAGE  = 'mrunal2804/hope-backend'
        FRONTEND_IMAGE = 'mrunal2804/hope-frontend'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test Backend') {
            steps {
                dir('backend') {
                    sh '''
                        python3 -m pip install -r requirements.txt
                        python3 -m pytest
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} ./backend
                    docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} ./frontend

                    docker tag ${BACKEND_IMAGE}:${BUILD_NUMBER} ${BACKEND_IMAGE}:latest
                    docker tag ${FRONTEND_IMAGE}:${BUILD_NUMBER} ${FRONTEND_IMAGE}:latest
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_PASSWORD" | docker login \
                            --username "$DOCKER_USER" \
                            --password-stdin

                        docker push ${BACKEND_IMAGE}:${BUILD_NUMBER}
                        docker push ${BACKEND_IMAGE}:latest
                        docker push ${FRONTEND_IMAGE}:${BUILD_NUMBER}
                        docker push ${FRONTEND_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI pipeline successful! Images pushed to Docker Hub.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins Console Output.'
        }
    }
}