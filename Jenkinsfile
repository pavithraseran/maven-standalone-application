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

        stage('Deploy to Artifactory - Classic') {
            steps {
                script {
                    rtServer(
                        id: "${ARTIFACTORY_SERVER}",
                        url: 'https://trialv00e9o.jfrog.io/artifactory/',
                        credentialsId: 'jfrog_user_pwd'
                    )

                    rtMavenDeployer(
                        id: 'Maven-Deployer',
                        serverId: "${ARTIFACTORY_SERVER}",
                        releaseRepo: 'libs-release-local',
                        snapshotRepo: 'libs-snapshot-local'
                    )

                    rtMavenRun(
                        tool: 'Maven-3.9.9',
                        pom: 'pom.xml',
                        goals: 'clean install',
                        deployerId: 'Maven-Deployer'
                    )

                    rtPublishBuildInfo(serverId: "${ARTIFACTORY_SERVER}")
                }
            }
        }
    }

    post {
        success {
            echo ' Classic: Build, analysis, and deployment completed!'
        }
        failure {
            echo ' Pipeline failed.'
        }
    }
}
