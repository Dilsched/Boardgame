pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE  = "boardgame-app"
        SONAR_PROJECT = "BoardgameListingWebApp"
    }

    stages {

        stage('Build') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test'
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
                    bat "mvn sonar:sonar -Dsonar.projectKey=%SONAR_PROJECT% -Dsonar.projectName=BoardGameApp"
                }
            }
        }

        stage('Security') {
            steps {
                bat 'if not exist dependency-check-report mkdir dependency-check-report'
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
                    bat 'docker rm -f boardgame-staging || exit 0'
                    bat "docker build -t boardgame-app:%BUILD_NUMBER% ."
                    bat "docker run -d --name boardgame-staging -p 8082:8080 boardgame-app:%BUILD_NUMBER%"
                }
            }
        }

        stage('Release') {
            steps {
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
                        file: "target/database_service_project-0.0.5-SNAPSHOT.jar",
                        type: 'jar'
                    ]]
                )
            }
        }

    }

    post {
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed"
        }
        always {
            cleanWs()
        }
    }
}
