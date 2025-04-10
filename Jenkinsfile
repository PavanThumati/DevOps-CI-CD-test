pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        IMAGE_NAME = "your-dockerhub-username/demo-app"
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
                    docker.withRegistry('', DOCKER_CREDENTIALS_ID) {
                        def app = docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}")
                        app.push('latest')
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
