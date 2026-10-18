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
                git branch: 'main',
                    url: 'https://github.com/Mrunal-2804/jenkins-ArgoCD-SQ.git'
            }
        }

        
        stage('Test Backend') {
            steps {
                sh '''
                    set -eu

                    TEST_CONTAINER="backend-test-${BUILD_NUMBER}"

                    cleanup() {
                        docker rm -f "$TEST_CONTAINER" >/dev/null 2>&1 || true
                    }
                    trap cleanup EXIT

                    docker create \
                        --name "$TEST_CONTAINER" \
                        python:3.13-slim \
                        sh -c 'cd /tmp/backend && python -m pip install -r requirements.txt pytest && python -m pytest'

                    docker cp "$WORKSPACE/backend" "$TEST_CONTAINER:/tmp/"

                    docker start --attach "$TEST_CONTAINER"
                '''
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

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'mySonar'

                    withSonarQubeEnv('MySonar') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dproject.settings=sonar-project.properties
                        """
                    }
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