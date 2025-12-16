pipeline {
    agent any

    environment {
        APP_NAME              = 'myapp'
        // For Docker Hub, image names are usually "username/repo"
        DOCKER_REPO           = 'ammujun29'
        DOCKER_IMAGE          = 'ammujun29/myapp-image'   // Explicit to avoid null
        // Uncomment if you use a non-default registry:
        // DOCKER_REGISTRY     = 'docker.io'
        DOCKER_CREDENTIALS_ID = 'docker-creds'
        // If you plan to use an NVD API key securely via Jenkins credentials:
        // NVD_API_KEY_CRED_ID = 'nvd-api-key'
    }

    tools {
        maven 'maven-3.9.11'
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

        stage('Dependency Check') {
            steps {
                sh 'mvn dependency-check:check'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:latest .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        echo "Logging in to Docker Hub..."
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                        echo "Pushing image ${DOCKER_IMAGE}:latest..."
                        docker push ${DOCKER_IMAGE}:latest

                        echo "Logout..."
                        docker logout
                    '''
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh '''
                    echo "Running Trivy scan on source code (non-blocking)..."
                    trivy fs \
                      --severity HIGH,CRITICAL \
                      --format table \
                      --output trivy-report.txt \
                      --scanners vuln \
                      --exit-code 0 \
                      .
                '''
                archiveArtifacts artifacts: 'trivy-report.txt', onlyIfSuccessful: false
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
    }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo "Pipeline finished for ${APP_NAME}. Cleaning            echo "Pipeline finished for ${APP_NAME}. Cleaning up / archiving artifacts if needed."
            // archiveArtifacts artifacts: 'target/*.jar', onlyIfSuccessful: true
        }
    }
}
