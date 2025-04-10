pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'DockerHubCredentials'
        IMAGE_NAME = "pavanthumati/java-app"
        REGISTRY = 'docker.io/pavanthumati'
        IMAGE_NAME = 'java-microservice'
        SONARQUBE_SERVER = 'Sonar-qube'
    }
pipeline {
  agent any
  triggers {
    githubPush()
  }

  tools {
    maven 'Maven 3' // Your Jenkins Maven installation
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test') {
      steps {
        sh 'mvn clean install'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("${SONARQUBE_SERVER}") {
          sh 'mvn sonar:sonar'
        }
      }
    }

    stage("Quality Gate") {
      when {
        not {
          branch 'feature/*'
        }
      }
      steps {
        timeout(time: 2, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Docker Build & Push') {
      when {
        branch 'develop'
      }
      steps {
        script {
          def imageTag = "${REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
          sh """
            docker build -t ${imageTag} .
            docker push ${imageTag}
          """
        }
      }
    }

    stage('Deploy to Kubernetes') {
      when {
        branch 'develop'
      }
      steps {
        sh """
          kubectl apply -f manifests/deployment.yaml
          kubectl apply -f manifests/service.yaml
        """
      }
    }
  }
}

}
