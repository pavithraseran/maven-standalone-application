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

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                            mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.organization=${SONAR_ORG} \
                            -Dsonar.host.url=https://sonarcloud.io \
                            -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to Artifactory') {
            steps {
                script {
                    rtServer(
                        id: "${ARTIFACTORY_SERVER}",
                        url: 'https://trial68cd6c.jfrog.io/artifactory/',
                        credentialsId: 'jfrog_new_token'
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

        stage('Deploy to Dev via Ansible') {
            steps {
                sh '''
                    ssh -i ~/downloads/ec2pemfile/pavi.pem -o StrictHostKeyChecking=no ubuntu@172.31.6.90 << 'EOF'
                      sudo su - ansible
                      echo "[INFO] Running Ansible playbook for DEV deployment..."
                      ansible-playbook /opt/deployment/ansible/deploy_app.yml \
                        -i /opt/deployment/ansible/inventory/dev/dev \
                        --vault-password-file /home/ansible/vault_pass.txt
                    EOF
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Build, SonarCloud scan, Artifactory upload, and Ansible deployment completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
    }
}
