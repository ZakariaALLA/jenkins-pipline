pipeline {
    agent any

    parameters {
        booleanParam(name: 'DEPLOY', defaultValue: false, description: 'Do you want to deploy this version ?')
    }

    environment {
        DOCKER_REGISTRY = 'zakariaalla'
        IMAGE_NAME = 'helloworld-java'
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN = credentials('jenkins-pipline-java-token')
        
        SHORT_COMMIT = bat """
        git rev-parse --short HEAD
        """
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
                    timeout(time: 5, unit: 'MINUTES') {
                        def qualityGate = waitForQualityGate()
                        if (qualityGate.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
                        }
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    bat """
                    docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${SHORT_COMMIT} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    bat """
                    docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${SHORT_COMMIT}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { return params.DEPLOY }
            }
            steps {
                script {
                    withEnv(["KUBECONFIG=C:/Users/D/.kube/config"]) {
                        bat """
                            kubectl delete pod hello-world-pod || echo "No existing pod found"
                            kubectl run hello-world-pod --image=${DOCKER_REGISTRY}/${IMAGE_NAME}:${SHORT_COMMIT} --port=8080
                            kubectl expose pod hello-world-pod --type=NodePort --name=hello-world-service
                        """
                    }
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
