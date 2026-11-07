pipeline {
    agent any

    environment {
        IMAGE_NAME = "srikantheggoju/inventory-app"
        MANIFEST_REPO = "https://github.com/Srikantheggoju/inventory-k8s-manifests.git"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Srikantheggoju/inventory-app.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                echo "Running SonarQube Scan"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                sh """
                git clone ${MANIFEST_REPO}
                cd inventory-k8s-manifests

                sed -i 's|image:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|g' deployment.yaml

                git config user.email "jenkins@local"
                git config user.name "Jenkins"

                git add .
                git commit -m "Updated image to ${BUILD_NUMBER}"
                git push origin main
                """
            }
        }
    }
}
