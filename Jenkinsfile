pipeline {
    agent any
    
    parameters{
        string(name : 'DOCKER_TAG', defaultValue: 'latest', description : 'docker_tag')
    }

    tools {
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'git-token',
                    url: 'https://github.com/chimdi247/Multi-Tier-BankApp-CI.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }

        stage('File System Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=bankapp \
                        -Dsonar.projectKey=bankapp \
                        -Dsonar.java.binaries=target
                    '''
                }
            }
        }
        
        stage('Build & Publish Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings-devopshark', maven: 'maven3', traceability: true) {
                      sh "mvn deploy -DskipTests=true"
               }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                            sh "docker build -t chimdi247/bankapp:$DOCKER_TAG ."
                  }
                }
            }
        }
        
         stage('Docker image Scan') {
            steps {
                sh "trivy image --format table -o image.html chimdi247/bankapp:$DOCKER_TAG"
            }
        }
        
         stage('Docker Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                            sh "docker push chimdi247/bankapp:$DOCKER_TAG"
                  }
                }
            }
        }
        
        stage('dummy') {
            steps {
                echo 'hi'
            }
        }
    }
}
