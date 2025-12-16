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
stage('SonarQube Analysis') {
            steps {
                // Ensure a SonarQube server is configured and the name matches here
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn -B sonar:sonar'
                }
            }
        }
 
        // Optional Quality Gate stage:
        // stage('Quality Gate') {
        //     steps {
        //         timeout(time: 10, unit: 'MINUTES') {
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }
 
        stage('Docker Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:latest .'
            }
        }
 
        stage('Docker Run') {
            steps {
                sh '''
                    docker rm -f ${APP_NAME} || true
                    docker run -d --name ${APP_NAME} -p 8082:8080 ${DOCKER_IMAGE}:latest
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
            // At least one real step is required — echo is safe.
            echo "Pipeline finished for ${APP_NAME}. Cleaning up / archiving artifacts if needed."
            // Example artifact archiving (uncomment if you want to use it):
            // archiveArtifacts artifacts: 'target/*.jar', onlyIfSuccessful: true
        }
    }
}
