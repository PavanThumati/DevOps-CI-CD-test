pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-qube')
        DOCKER_CREDENTIALS_ID = 'DockerHubCredentials'
        IMAGE_NAME = "pavanthumati/java-app"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('SonarQube Analysis') {
            when {
                not {
                    branch 'main'
                }
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }

        stage('Docker Build & Push') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    withDockerRegistry([credentialsId: 'DockerHubCredentials', url: 'https://index.docker.io/v1/']) {
                        sh 'docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} .'
                        sh 'docker push ${IMAGE_NAME}:${env.BUILD_NUMBER}'
                    }
                }
            }
        }

        stage('Kubernetes Deploy') {
            when {
                branch 'develop'
            }
            steps {
                sh 'kubectl apply -f manifests/'
            }
        }
    }
}
