pipeline {
    agent any

    environment {
        IMAGE_NAME = "nadafouuad/flask-app"
        CONTAINER_NAME = "flask-app-container"
    }

    stages {

        stage('Checkout Repo') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-app .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Tag Image') {
            steps {
                sh """
                    docker tag flask-app \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
                """
            }
        }

        stage('Push Image') {
            steps {
                sh """
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                """
            }
        }

        stage('Run Container') {
            steps {
                sh """
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    -p 5000:5000 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
                """
            }
        }
    }

    post {
        success {
            echo "Docker deployment successful!"
            echo "Image: ${IMAGE_NAME}:${BUILD_NUMBER}"
        }

        failure {
            echo "Docker pipeline failed!"
        }
    }
}