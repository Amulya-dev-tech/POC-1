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

        stage('Trivy FS Scan') {
            steps {
                sh '''
                    trivy fs \
                    --severity HIGH,CRITICAL \
                    --exit-code 1 \
                    .
                '''
            }
        }

        stage('Push Image') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                echo "Pushing image to registry..."
                // Example:
                // sh 'docker push ${DOCKER_IMAGE}:latest'
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
            // archiveArtifacts artifacts: 'target/*.jar', onlyIfSuccessful: true
        }
       }
