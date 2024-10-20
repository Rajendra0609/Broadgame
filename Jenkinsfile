pipeline {
    agent any
    tools {
        maven 'maven'
    }
    parameters {
    string(name: 'ENVIRONMENT', defaultValue: 'development', description: 'Choose the environment for deployment')
    booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip tests during build?')
    string(name: 'IMAGE_NAME', defaultValue: 'daggu1997/broadgame', description: 'Docker image name for the Kubernetes deployment')
     string(name: 'TAG_VERSION', defaultValue: 'latest', description: 'Image tag version')
    string(name: 'REPLICAS', defaultValue: '1', description: 'Number of replicas for the deployment')
}
    environment {
        SCANNER_HOME = tool 'sonarqube'
        IMAGE_NAME = env.IMAGE_NAME
        IMAGE_TAG = env.TAG_VERSION
    }
    stages {
        stage('Preparation') {
            steps {
                script {
                    echo "Building for environment: ${params.ENVIRONMENT}"
                }
            }
        }
        stage('Maven Clean and Compile') {
            parallel {
                stage('Maven Clean') {
                    steps {
                        script {
                            try {
                                sh 'mvn clean'
                            } catch (Exception e) {
                                error("Maven Clean failed: ${e.message}")
                            }
                        }
                    }
                }
                stage('Maven Compile') {
                    steps {
                        script {
                            try {
                                sh 'mvn compile'
                            } catch (Exception e) {
                                error("Maven Compile failed: ${e.message}")
                            }
                        }
                    }
                }
            }
        }
        stage('Install') {
            steps {
                script {
                    try {
                        sh 'mvn package'
                        archiveArtifacts artifacts: 'target/*.jar'
                    } catch (Exception e) {
                        error("Maven Install failed: ${e.message}")
                    }
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    if (!params.SKIP_TESTS) {
                        try {
                            sh 'mvn test'
                            sh 'mvn integration-test'
                        } catch (Exception e) {
                            error("Maven Test failed: ${e.message}")
                        }
                    } else {
                        echo "Skipping tests as per user input."
                    }
                }
            }
        }
        stage('Lynis Security Scan') {
            steps {
                script {
                    try {
                        sh 'lynis audit system | ansi2html > lynis-report.html'
                        echo "Lynis report path: ${env.WORKSPACE}/lynis-report.html"
                        archiveArtifacts artifacts: 'lynis-report.html'
                    } catch (Exception e) {
                        error("Lynis Security Scan failed: ${e.message}")
                    }
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                script {
                    try {
                        dependencyCheck(
                            additionalArguments: '--scan ./ --format HTML',
                            odcInstallation: 'dpcheck'
                        )
                        archiveArtifacts(
                            artifacts: '**/dependency-check-report.html',
                            allowEmptyArchive: true
                        )
                    } catch (Exception e) {
                        error("OWASP FS SCAN failed: ${e.message}")
                    }
                }
            }
        }
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Broadgame \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Broadgame \
                        -Dsonar.login=raja \
                        -Dsonar.password=raja
                    '''
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    try {
                        sh "mvn package -DskipTests=${params.SKIP_TESTS}"
                    } catch (Exception e) {
                        error("Maven Build failed: ${e.message}")
                    }
                }
            }
        }
        stage('Approval') {
            steps {
                script {
                    // Approval step from admin
                    def approval = input(
                        id: 'Approval', 
                        message: 'Do you want to proceed with building the Docker image?',
                        parameters: [
                            [$class: 'BooleanParameterDefinition', name: 'Proceed', defaultValue: true]
                        ]
                    )
                    if (!approval) {
                        error("Build was not approved by admin.")
                    }
                }
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    try {
                        def version = "${env.BUILD_NUMBER}-${params.ENVIRONMENT}" // Use build number and environment for versioning
                        withDockerRegistry(credentialsId: 'dockerhub'){
                            sh "docker build -t ${params.IMAGE_NAME}:${params.IMAGE_TAG} ."
                        }
                    } catch (Exception e) {
                        error("Docker Build failed: ${e.message}")
                    }
                }
            }
        }
        stage('TRIVY') {
            steps {
                script {
                    try {
                        sh "trivy image --format table --timeout 15m -o trivy-image-report.html ${params.IMAGE_NAME}:${params.IMAGE_TAG}"
                        echo "Trivy report path: ${env.WORKSPACE}/trivy-image-report.html"
                        archiveArtifacts artifacts: 'trivy-image-report.html'
                    } catch (Exception e) {
                        error("TRIVY failed: ${e.message}")
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    try {
                        withDockerRegistry(credentialsId: 'dockerhub') {
                            sh "docker push ${params.IMAGE_NAME}:${params.IMAGE_TAG}"
                        }
                    } catch (Exception e) {
                        error("Docker Push failed: ${e.message}")
                    }
                }
            }
        }
        stage('Update the tag for Yaml file') {
        steps {
        script {
                    // Approval step from admin
                    def approval = input(
                        id: 'Approval', 
                        message: 'Do you want to proceed with building the Docker image?',
                        parameters: [
                            [$class: 'BooleanParameterDefinition', name: 'Proceed', defaultValue: true]
                        ]
                    )
                    if (!approval) {
                        error("Build was not approved by admin.")
                    }
                }
    }
}
        stage('Update Image Tag in YAML') {
    steps {
        script {
            // Extract the image tag
            def imageTag = sh(script: "grep -oP '(?<=daggu1997/broadgame:)[^ ]+' deployment-service.yaml", returnStdout: true).trim()

            // Update the deployment-service.yaml with the new image tag
            sh "sed -i 's|:.*|:${imageTag}|g' deployment-service.yaml"
        }
    }
}
    stage('Deplay the Yaml file to kubernetes') {
        steps {
        script {
                    // Approval step from admin
                    def approval = input(
                        id: 'Approval', 
                        message: 'Do you want to proceed with building the Docker image?',
                        parameters: [
                            [$class: 'BooleanParameterDefinition', name: 'Proceed', defaultValue: true]
                        ]
                    )
                    if (!approval) {
                        error("Build was not approved by admin.")
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
