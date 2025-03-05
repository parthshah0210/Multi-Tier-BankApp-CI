pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    environment{ 
        SCANNER_HOME = tool 'sonarqube'
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }
        stages {
        stage('code checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/parthshah0210/Multi-Tier-BankApp-CI.git'
            }
        }
        
        stage('code compile') {
            steps {
                sh "mvn clean compile -U"
            }
        }
       
        stage('code test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('File scan') {
            steps {
                sh "trivy fs --format table -o filescan-trivy.html ."
            }
        }
       
        stage('sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=bankapp -Dsonar.projectKey=bankapp \
                            -Dsonar.java.binaries=target '''    
                }
            }
        }
        stage('code Qualitygate') {
            steps {
               timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonar-cred'
                } 
            }
        }
        stage('code build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('code deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-file', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('build and tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred') {
                        sh 'docker build -t parthu210/bankapp:$IMAGE_TAG .'
                    } 
                }
            }
        }
         stage('Image Scan') {
            steps {
                sh "trivy image --format table -o filescan-trivy.html parthu210/bankapp:$IMAGE_TAG"
            }
        }
        stage('Docker Image Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred') {
                        sh 'docker push parthu210/bankapp:$IMAGE_TAG'
                    } 
                }
            }
        }
    }
}
