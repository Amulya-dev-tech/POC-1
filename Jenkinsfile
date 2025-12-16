pipeline {
    agent any
 
    environment {
        APP_NAME     = 'myapp'
        DOCKER_IMAGE = 'myapp-image'
        // DOCKER_REGISTRY = 'registry.example.com'
        // DOCKER_CREDENTIALS_ID = 'docker-creds'
    }
 
    tools {
        // Change this to match the name configured in Jenkins Global Tool Configuration.
        // Either set up a tool named "maven-3.9.11" in Jenkins, or change this name below.
        maven 'maven-3.9.11'
        // jdk 'jdk-17'
    }
 
    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/Amulya-dev-tech/POC-1.git'
            }
        }
 
        stage('Maven Build') {
            steps {
                sh 'mvn -V -B clean package -DskipTests'
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
            // At least one real step is required — echo is safe.
            echo "Pipeline finished for ${APP_NAME}. Cleaning up / archiving artifacts if needed."
            // Example artifact archiving (uncomment if you want to use it):
            // archiveArtifacts artifacts: 'target/*.jar', onlyIfSuccessful: true
        }
    }
}
