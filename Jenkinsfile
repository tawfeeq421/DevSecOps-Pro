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
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/tawfeeq421/DevSecOps-Pro.git'
            }
        }

        stage("Sonarqube Analysis ") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix -Dsonar.projectKey=Netflix"
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
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
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
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                        sh "docker build --build-arg TMDB_V3_API_KEY=975ae9f47f4e37a3c9247707a2e342c4 -t netflix ."
                        sh "docker tag netflix tawfeeq421/netflix:v1 "
                        sh "docker push tawfeeq421/netflix:v1 "
                    }
                }
            }
        }

        stage("TRIVY") {
            steps {
                sh "trivy image tawfeeq421/netflix:v1 > trivyimage.txt" 
            }
        }

        stage('Deploy to container') {
            steps {
                sh 'docker run -d --name netflix -p 8081:80 tawfeeq421/netflix:v1'
            }
        }
        
    }
}
