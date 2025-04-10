pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'DockerHubCredentials'
        IMAGE_NAME = "pavanthumati/java-app"
        REGISTRY = 'docker.io/pavanthumati'
        IMAGE_NAME = 'java-microservice'
        SONARQUBE_SERVER = 'sonar-qube'
    }
 pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'sonar-qube'
        DOCKER_IMAGE = 'pavanthumati/hello-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                    expression { env.BRANCH_NAME.startsWith('feature/') }
                }
            }
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Docker Build & Push') {
            when {
                branch 'develop'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'DockerHubCredentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        docker login -u $DOCKER_USER -p $DOCKER_PASS
                        docker build -t $DOCKER_IMAGE .
                        docker push $DOCKER_IMAGE
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'develop'
            }
            steps {
                sh 'kubectl apply -f manifests/'
            }
        }
    }
}
