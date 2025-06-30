pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/santoshbd67/Hotstar-Clone.git'
            }
        }

        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Hotstar \
                        -Dsonar.projectKey=Hotstar'''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh """#!/bin/bash
                        dependency-check.sh \
                            --nvdApiKey \$NVD_API_KEY \
                            --scan ./ \
                            --format XML \
                            --disableYarnAudit \
                            --disableNodeAudit \
                            --out .
                        """
                        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                    }
                }
            }
        }

        stage('Docker Scout FS') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker-scout quickview fs://.'
                        sh 'docker-scout cves fs://.'
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker build -t hotstar .'
                        sh 'docker tag hotstar santoshbd67/hotstar:latest'
                        sh 'docker push santoshbd67/hotstar:latest'
                    }
                }
            }
        }

        stage('Docker Scout Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker-scout quickview santoshbd67/hotstar:latest'
                        sh 'docker-scout cves santoshbd67/hotstar:latest'
                        sh 'docker-scout recommendations santoshbd67/hotstar:latest'
                    }
                }
            }
        }

        stage('deploy_docker') {
            steps {
                sh 'docker run -d --name hotstar -p 3000:3000 santoshbd67/hotstar:latest'
            }
        }
    }
}
