pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE     = "boardgame-app"
        DOCKER_TAG       = "${BUILD_NUMBER}"
        SONAR_PROJECT    = "BoardgameListingWebApp"
        NEXUS_VERSION    = "nexus3"
        NEXUS_PROTOCOL   = "http"
        NEXUS_URL        = "localhost:8081"
        NEXUS_REPO       = "maven-releases"
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Code Quality') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT} \
                          -Dsonar.projectName='BoardGame Listing Web App'
                    """
                }
            }
        }

        stage('Security') {
            steps {
                dependencyCheck(
                    additionalArguments: '--scan target/ --format HTML --format XML --out dependency-check-report',
                    odcInstallation: 'OWASP-DC'
                )
            }
            post {
                always {
                    dependencyCheckPublisher pattern: 'dependency-check-report/dependency-check-report.xml'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    bat 'docker rm -f boardgame-staging || true'
                    bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
                    bat "docker run -d --name boardgame-staging -p 8082:8080 %DOCKER_IMAGE%:%DOCKER_TAG%"
                }
            }
        }

        stage('Release') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: 'localhost:8081',
                        groupId: 'com.javaproject',
                        version: "0.0.${BUILD_NUMBER}",
                        repository: 'maven-releases',
                        credentialsId: 'nexus-credentials',
                        artifacts: [[
                            artifactId: 'database_service_project',
                            classifier: '',
                            file: "target/database_service_project-0.0.1-SNAPSHOT.jar",
                            type: 'jar'
                        ]]
                    )
                }
            }
        }

    }

    post {
        success {
            mail(
                to: 'team@example.com',
                subject: "BUILD #${BUILD_NUMBER} PASSED",
                body: "All stages completed successfully.\nBuild: ${BUILD_URL}"
            )
        }
        failure {
            mail(
                to: 'team@example.com',
                subject: "BUILD #${BUILD_NUMBER} FAILED",
                body: "A stage has failed. Please review: ${BUILD_URL}"
            )
        }
        always {
            cleanWs()
        }
    }
}