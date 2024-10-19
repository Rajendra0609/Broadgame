pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        SCANNER_HOME = tool 'sonarqube_1'
    }
    stages {
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube_1') {
                        sh '''
                           $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=webapplication_ekart \
                           -Dsonar.java.binaries=. \
                           -Dsonar.projectKey=webapplication_ekart \
                           -Dsonar.login=$sonar
                        '''
                  }
                }
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh "docker build -t daggu1997/broadgame:latest ."
                    }
                }
            }
        }
        stage('TRIVY') {
            steps {
                script {
                    // Perform security scan of the image and write output to an HTML file
                    sh 'trivy image --format table --timeout 5m -o trivy-image-report.html daggu1997/broadgame:latest'

                    // Display the absolute path of the report in the Jenkins console output
                    def reportPath = "${env.WORKSPACE}/trivy-image-report.html"
                    echo "Trivy report path: ${reportPath}"

                    // Archive the report file for access after the build
                    archiveArtifacts artifacts: 'trivy-image-report.html'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh "docker push daggu1997/broadgame:latest"
                    }
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
