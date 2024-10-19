pipeline {
    agent any
    tools{
        maven 'maven'
    }
    environment {
        SCANNER_HOME=tool 'sonarqube'
    }
    stages {
        stage('mvn clean') {
            steps{
                sh 'mvn clean'
                sh 'mvn validate'
        }
        stage('mvn compile') {
            steps{
                sh 'mvn clean compile'
            }
        }
        stage('install'){
            steps{
                sh 'mvn package'
               archiveArtifacts artifacts: 'target/*.jar' 
            }
        }
        stage('test'){
            steps{
                sh 'mvn test'
                sh 'integration-test'
            }
        }
        stage('Lynis Security Scan') {
            steps {
                script {
                    // Execute the Lynis security scan and convert the output to HTML
                    sh 'lynis audit system | ansi2html > lynis-report.html'

                    // Display the absolute path of the report in the Jenkins console output
                    def reportPath = "${env.WORKSPACE}/lynis-report.html"
                    echo "Chemin du rapport Lynis : ${reportPath}"

                    // Archive the report file for access after the build
                    archiveArtifacts artifacts: 'lynis-report.html'
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                script {
                    Run OWASP Dependency Check
                   dependencyCheck additionalArguments: '--scan ./ --format HTML', odcInstallation: 'dpcheck'
                
                    Publish the Dependency Check report
                  dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                    archiveArtifacts artifacts: '**/dependency-check-report.html', allowEmptyArchive: true
                }
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonarqube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=webapplication_ekart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=webapplication_ekart '''
                }
            }
        }
        stage('Build'){
            steps{
                sh "mvn package -DskipTests=true "
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
                    // Effectue le balayage de sécurité de l'image et écrit la sortie dans un fichier HTML
                    sh 'trivy image --format table --timeout 5m -o trivy-image-report.html daggu1997/broadgame:latest'

                    // Affiche le chemin absolu du fichier de rapport dans la console de sortie Jenkins
                    def reportPath = "${WORKSPACE}/trivy-image-report.html"
                    echo "Chemin du rapport Trivy : ${reportPath}"

                    // Archive le fichier de rapport pour qu'il soit accessible après la construction
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
                sh """
                echo "Cleaned Up Workspace For Project"
                """
            }
        }
    }
}
