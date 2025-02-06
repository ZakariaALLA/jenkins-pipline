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
        stage('Get Commit Hash') {
            steps {
                script {
                    def commitOutput = bat(script: 'git rev-parse --short HEAD', returnStdout: true)
                    def lines = commitOutput.split("\r?\n")
                    def commit = lines[lines.length - 1].trim()

                    echo "Extracted Commit Hash: ${commit}"
                    env.SHORT_COMMIT = commit
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat """
                        docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.SHORT_COMMIT} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-hub-credentials', url: 'https://index.docker.io/v1/']) {
                        bat "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.SHORT_COMMIT}"
                    }
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
                            echo "Deleting existing job if it exists..."
                            kubectl delete job hello-world-job --ignore-not-found=true

                            echo "Creating a Kubernetes Job for Hello World App..."
                            kubectl apply -f - <<EOF
                            apiVersion: batch/v1
                            kind: Job
                            metadata:
                            name: hello-world-job
                            spec:
                            template:
                                spec:
                                containers:
                                - name: hello-world
                                    image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.SHORT_COMMIT}
                                    command: ["java", "Hello"]
                                restartPolicy: Never
                            EOF
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