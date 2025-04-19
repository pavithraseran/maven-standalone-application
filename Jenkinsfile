pipeline { 
    agent any 
        tools { 
        maven 'Maven-3.9.9'
         jdk 'JDK-17'   
        }
        environment {
        SONARQUBE = 'SonarCloud'  // Use the name you gave SonarCloud in Jenkins config
        SONAR_PROJECT_KEY = 'pavithraseran'  // Replace with your project key
        SONAR_ORG = 'pavithraseran'  // Replace with your SonarCloud organization name
        ARTIFACTORY_SERVER = 'Jfrog_Artifactory'  // Replace with your Artifactory server ID configured in Jenkins

    }
        stages {
            stage ('vcs')
            {
                steps {
                    git url : 'https://github.com/pavithraseran/maven-standalone-application.git' , branch : 'master'
                }
                
            }
                stage ('build')
                {
                    steps {
                        sh " mvn clean package "
                    }
                    
                }
                stage ('sonar analysis')
                {
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
            
stage('Deploy to Artifactory') {
            steps {
                script {
                    rtServer(
                        id: "${ARTIFACTORY_SERVER}",
                        url: 'https://trialv00e9o.jfrog.io/artifactory/',
                        credentialsId: 'jenkins-jfrog-token'
                    )

                    rtMavenDeployer(
                        id: 'Maven-Deployer',
                        serverId: "${ARTIFACTORY_SERVER}",
                        releaseRepo: 'libs-release-local',  // Use your release repository name
                        snapshotRepo: 'libs-snapshot-local'  // Optional: if you use snapshots
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
            echo 'Build, analysis, and deployment completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
