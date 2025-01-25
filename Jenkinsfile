pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'zakariaalla'
        IMAGE_NAME = 'helloworld-java'
        VERSION = '1.0'
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN = credentials('jenkins-pipline-java-token')
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
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('LocalSonarQube') {
                    bat """
                    sonar-scanner -Dsonar.projectKey=jenkins-pipline -Dsonar.sources=. -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_AUTH_TOKEN}
                    """
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
                    }
                }
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
