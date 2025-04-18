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
        }
        post {
        success {
            echo 'Build and SonarCloud Analysis complete!'
        }
        failure {
            echo 'Build failed.'
}
}
}

