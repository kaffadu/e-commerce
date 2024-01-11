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
                git branch: 'main', credentialsId: '60c1e597-70d2-455c-884e-0298d4d7bbee', url: 'https://github.com/kaffadu/ecommerce-app.git'
            }
        }
        
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=ecommerce-app \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=ecommerce-app '''
                }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('Docker Build and Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'e28f1a84-8990-460c-86c6-7961c6eca08e', toolName: 'docker') {
                        
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag  shopping-cart kaffadu/shopping-cart:latest"
                        sh "docker push kaffadu/shopping-cart:latest"
                    }
                } 
            }
        }
        
