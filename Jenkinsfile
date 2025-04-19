pipeline {
    agent any

    tools {
        maven 'Maven-3.9.9'
        jdk 'JDK-17'
    }

    environment {
        SONARQUBE = 'SonarCloud'
        SONAR_PROJECT_KEY = 'pavithraseran'
        SONAR_ORG = 'pavithraseran'
        ARTIFACTORY_SERVER = 'Jfrog_Artifactory'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/pavithraseran/maven-standalone-application.git', branch: 'master'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                            mvn sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.organization=${SONAR_ORG} \
                            -Dsonar.host.url=https://sonarcloud.io \
                            -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Deploy to Artifactory - Modern') {
            steps {
                script {
                    def server = Artifactory.server("${ARTIFACTORY_SERVER}")

                    def deployer = Artifactory.newMavenDeployer()
                    deployer.server = server
                    deployer.releaseRepo = 'libs-release-local'
                    deployer.snapshotRepo = 'libs-snapshot-local'

                    def buildInfo = Artifactory.newBuildInfo()

                    deployer.deployPom = true
                    deployer.deployArtifacts = true

                    deployer.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo

                    server.publishBuildInfo buildInfo
                }
            }
        }
    }

    post {
        success {
            echo ' Modern: Build, analysis, and deployment completed!'
        }
        failure {
            echo ' Pipeline failed.'
        }
    }
}
