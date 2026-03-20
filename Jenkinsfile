pipeline {
    agent any

    environment {
        GIT_REPO = "https://github.com/veeravenkateswararao/react.git"
        GIT_BRANCH = "main"

        DOCKERHUB_USER = "venkyveera"
        IMAGE_NAME = "react-app"
        IMAGE_TAG = "${BUILD_NUMBER}"

        DOCKER_CREDS = "Docker_CRED"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scmGit(
                    branches: [[name: "*/${GIT_BRANCH}"]],
                    userRemoteConfigs: [[
                        credentialsId: 'venkygit',
                        url: "${GIT_REPO}"
                    ]]
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    """
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh """
                docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl apply -f k8s/
                kubectl set image deployment/venky-react-deploy \
                venky-react-cont=${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}

                kubectl rollout status deployment/venky-react-deploy
                """
            }
        }
    }
}
