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
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/GopichandKosuri739/Netflix-Clone-DevSecOps-Project.git'
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Netflix \
                        -Dsonar.projectKey=Netflix'''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./src --noupdate --disableYarnAudit --disableNodeAudit --exclude node_modules --format ALL', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                sh 'echo $USER'
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        withCredentials([string(credentialsId: 'TMDB_V3_API_KEY', variable: 'TMDB_API')]) {
                            sh '''#!/bin/bash
                                docker build --build-arg TMDB_V3_API_KEY=${TMDB_API} -t netflix .
                                docker tag netflix gopichand7391/netflix:latest
                                docker push gopichand7391/netflix:latest
                                echo "✅ Docker image pushed into Docker Hub successfully"
                            '''
                        }
                    }
                }
            }
        }

        stage("TRIVY") {
            steps {
                sh "trivy image gopichand7391/netflix:latest > trivyimage.txt"
            }
        }

        stage('Deploy to container') {
            steps {
                sh 'docker run -d --name netflix-clone -p 8082:80 gopichand7391/netflix:latest'
                sh 'echo Hurry!! Netflix Container Deployed Successfully'
            }
        }
    }
post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'gopichandkosuri739@gmail.com',                              
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}
