pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        SCANNER_HOME = tool 'sonarqube_1'
        SONAR_LOGIN_TOKEN = credentials('sonar') // Reference the Jenkins credential ID
    }
    stages {
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube_1') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=webapplication_ekart \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=webapplication_ekart \
                        -Dsonar.login=$SONAR_LOGIN_TOKEN
                    '''
                }
            }
        }
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
                sh 'echo "Cleaned Up Workspace For Project"'
            }
        }
    }
}
