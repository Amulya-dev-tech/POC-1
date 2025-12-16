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
        maven 'Maven-3.9.11'
        // jdk 'jdk-17'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/Amulya-dev-tech/POC-10.git'
            }
        }

        stage('Build & Test (with coverage)') {
            steps {
                // Runs tests and JaCoCo report because your POM binds:
                // - jacoco:prepare-agent (default)
                // - jacoco:report at "verify"
                sh 'mvn -V -B clean verify'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            mvn -B sonar:sonar \
                                -Dsonar.projectKey=gs-spring-boot \
                                -Dsonar.projectName=gs-spring-boot \
                                -Dsonar.sources=src \
                                -Dsonar.tests=test \
                                -Dsonar.java.binaries=target/classes \
                                -Dsonar.junit.reportPaths=target/surefire-reports \
                                -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \
                                -Dsonar.sourceEncoding=UTF-8 \
                                -Dsonar.login=${SONAR_TOKEN} \
                                ${SONAR_HOST_URL:+-Dsonar.host.url=${SONAR_HOST_URL}}
                        '''
                    }
                }
            }
        }

        // Optional Quality Gate stage:
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
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
            //            // archiveArtifacts artifacts: 'target/*.jar', onlyIfSuccessful: true
        }
    }
