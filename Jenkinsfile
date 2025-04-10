pipeline {
    agent any

    environment {
        IMAGE_NAME = 'pavanthumati/java-hello-app'
        TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
                sh 'ls -l target'

            }
        }

        stage('Docker Build and Push') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'DockerHubCredentials') {
                        def image = docker.build("${IMAGE_NAME}:${TAG}")
                        image.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'develop'
            }
            steps {
                 withKubeConfig([credentialsId: 'kubeconfig'])
                {
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
}
