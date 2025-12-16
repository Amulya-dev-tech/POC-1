
pipeline {
  agent any

  environment {
    APP_NAME     = 'myapp'
    DOCKER_IMAGE = 'myapp-image'
  }

  tools {
    // Must match the name configured in "Manage Jenkins > Global Tool Configuration"
    maven 'maven-3.9.11'
  }

  stages {
    stage('Git Clone') {
      steps {
        git branch: 'main', url: 'https://github.com/Amulya-dev-tech/POC-1.git'
      }
    }

    stage('Build & Test (with coverage)') {
      steps {
        // Runs unit tests; JaCoCo XML report generated at target/site/jacoco/jacoco.xml during "verify"
        sh 'mvn -V -B clean verify'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        // Injects SONAR_HOST_URL & credentials from Jenkins "Configure System"
        withSonarQubeEnv('SonarQube') {
          sh '''
            mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar \
              -Dsonar.projectKey=gs-spring-boot \
              -Dsonar.projectName=gs-spring-boot \
              -Dsonar.sourceEncoding=UTF-8 \
              -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
          '''
        }
      }
    }

    stage('Quality Gate') {
      steps {
        Requires SonarQube webhook: http://13.204.90.126:8080/sonarqube-webhook/
        timeout(time: 10, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t ${DOCKER_IMAGE}:latest .'
      }
    }

    stage('Docker Run') {
      steps {
        sh '''
          docker rm -f ${APP_NAME} || true
          docker run -d --name ${APP_NAME} -p 8081:8080 ${DOCKER_IMAGE}:latest
        '''
      }
    }
  }

  post {
    success {
      echo 'Pipeline executed successfully!'
    }
    failure {
      echo 'Pipeline failed!'
    }
    always {
      echo "Pipeline finished for ${APP_NAME}. Cleaning up / archiving artifacts if needed."
      // archiveArtifacts artifacts: 'target/*.      // archiveArtifacts artifacts: 'target/*.jar', onlyIfSuccessful: true
    }
  }
