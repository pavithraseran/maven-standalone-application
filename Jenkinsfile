pipeline {
    agent any

    tools {
        maven 'Maven-3.9.9'
        jdk 'JDK-17'
    }

    environment {
        SONARQUBE = 'SonarCloud'
        SONAR_PROJECT_KEY = 'myapp-key'
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'your_git_creds_id', url: 'https://your.repo.url/myapp.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Sonar Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    // FIXED: Use double-quotes so ${SONAR_PROJECT_KEY} is interpolated
                    sh "mvn sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY}"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to MN1 using Ansible') {
            steps {
                sshagent (credentials: ['your_ssh_key_id']) {
                    sh '''
                        ansible-playbook -i /opt/deployment/ansible/inventory/dev \
                        /opt/deployment/ansible/deploy_app.yml \
                        --vault-password-file /home/ansible/vault_pass.txt
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed!'
        }
        success {
            echo 'Deployment successful!'
        }
    }
}
