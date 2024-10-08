pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16' // Ensure the correct version is used
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner' // Correct reference to SonarQube scanner tool
    }
    stages {
        stage("Git Pull") {
            steps {
                git branch: 'main', url: 'https://github.com/chiragdcmarix/Uptime-kuma.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') { // Ensure correct SonarQube server
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Chatbot \
                    -Dsonar.projectKey=Chatbot '''
                }
            }
        }
        stage('Sonar-quality-gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
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
                sh "trivy fs . > trivyfs.json"
            }
        }
        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t uptime ."
                        sh "docker tag uptime jinesh1893/uptime:latest"
                        sh "docker push jinesh1893/uptime:latest"
                    }
                }
            }
        }
        stage("TRIVY") {
            steps {
                sh "trivy image jinesh1893/uptime:latest > trivy.json"
            }
        }
        stage("Remove container") {
            steps {
                sh "docker stop uptime || true"
                sh "docker rm uptime || true"
            }
        }
        stage('Deploy to container') {
            steps {
                sh 'docker run -d --name uptime -v /var/run/docker.sock:/var/run/docker.sock -p 3010:3010 jinesh1893/uptime:latest'
            }
        }
    }
} // Correct closing of pipeline block
