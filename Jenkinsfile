pipeline {
    agent any

    environment {
        APP_NAME              = 'myapp'
        DOCKER_REPO           = 'ammujun29'
        DOCKER_IMAGE          = 'ammujun29/myapp-image'
        DOCKER_CREDENTIALS_ID = 'docker-creds'
        // Securely load NVD API key from Jenkins credentials (Secret Text recommended)
        NVD_API_KEY           = credentials('NVD_API_KEY')
    }

    tools {
        // Ensure these tool names match Jenkins Global Tool Configuration
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
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./app/backend --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        post {
                always {
                    archiveArtifacts artifacts: 'report/**', allowEmptyArchive: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:latest .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
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
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.txt', allowEmptyArchive: true
                }
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
            // Example: archive build outputs when available
            // archiveArtifacts artifacts: 'target/*.jar', onlyIfSuccessful: true
        }
    }
}
