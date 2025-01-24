pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'zakariaalla'
        IMAGE_NAME = 'helloworld-java'
        VERSION = '1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'This is a Build stage From a minimal pipeline.' 
            }
        }
        stage('Unit Tests') { 
            steps { 
               echo 'This is a Unit Tests run stage.' 
            }
        }
        stage('Sonar') { 
            steps { 
               echo 'This is a sonar check stage.' 
            }
        }
        stage('Gate Way') { 
            steps { 
               echo 'This is a gateway check stage.' 
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    bat """
                    docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "This stage is to push the image ${DOCKER_REGISTRY}/${IMAGE_NAME}:${VERSION} to Artifactory."
                }
            }
        }
    }

    post {
        always {
            echo 'Build is finished!'
        }
        success {
            echo 'Build and push successful!'
        }
        failure {
            echo 'Build or push failed.'
        }
    }
}
