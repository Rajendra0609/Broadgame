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
                        withCredentials([string(credentialsId: 'sonar_user', variable: 'SONAR_TOKEN')]) {
                        sh '''
                        /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonarqube/bin/sonar-scanner \
                        -Dsonar.projectName=game \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=game \
                        -Dsonar.login=${SONAR_TOKEN}
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
