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
        // If your SonarQube server is configured in Jenkins "Configure System" with a token,
        // withSonarQubeEnv('<Name>') injects SONAR_HOST_URL and auth automatically.
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
        timeout(time: 10, unit: 'MINUTES') {
          Requires SonarQube webhook -> http://13.204.90.126:8080/sonarqube-webhook/
          waitForQualityGate abortPipeline: true
        }
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
      // archiveArtifacts artifacts: 'target/*.jar', onlyIfSuccessful: true
    }
  }
}
