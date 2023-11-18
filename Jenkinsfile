pipeline {
    agent any
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/prakashbelote/Ekart.git'        
                }
        }
        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        stage('OWASP Scan') {
            steps {
                 dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        stage('Trivi File Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Ekart-Demo \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Ekart-Demo '''
               }
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Docker Image Build') {
            steps {
                script{
                    withDockerRegistry(credentialsId:  '70e0ce70-b8c5-4aba-b853-4f74ced373a9', toolName: 'docker') {
                        
                       sh "docker build -t shopping-cartnew -f docker/Dockerfile ."
                        sh "docker tag  shopping-cartnew prakashbelote/shopping-cartnew:latest"
                       
                    }
                }
            }
        }
        stage('Trivi Image Scan') {
            steps {
                sh "trivy image  prakashbelote/shopping-cartnew:latest"
            }
        }
        stage('Docker Image Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId:  '70e0ce70-b8c5-4aba-b853-4f74ced373a9', toolName: 'docker') {
                        
                       sh "docker push prakashbelote/shopping-cartnew:latest"
                    }
                }
            }
        }
        stage('Deploy to Container') {
            steps {
                script{
                    withDockerRegistry(credentialsId:  '70e0ce70-b8c5-4aba-b853-4f74ced373a9', toolName: 'docker') {
                        
                      sh "docker run -d -p 5000:8070 prakashbelote/shopping-cartnew:latest"
                    }
                }
            }
        }
        
    }
}
