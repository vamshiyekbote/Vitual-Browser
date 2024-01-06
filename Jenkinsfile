pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
            stage('Git-Checkout') {
                steps {
                git branch: 'main', url: 'https://github.com/vamshiyekbote/Vitual-Browser.git'
                }
            }

            stage('Owasp-Dependency-Check-for-Vulnerabilities') {
                steps {
                dependencyCheck additionalArguments: '--scan ./ --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }

            stage('SonarQube-for-Code-Review') {
                steps {
                    withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Virtual_Browser \
                    -Dsonar.projectName=Virtual_Browser '''
                    }
                }
            }
            stage('Docker-Build & Image-Tag') {
                steps {
                    script {
                        withDockerRegistry(credentialsId: 'docker-cred', toolName: 'Docker') {
                            dir('/home/ubuntu/.jenkins/workspace/Virtual-Browser/.docker/firefox') {
                            sh 'docker build -t vamshiyekbote/virtualbrowser:new-build .'
                            }
                        }
                    }
                }
            }

            stage('Trivy-For-Docker-Image-Scan') {
                steps {
                    sh 'trivy image vamshiyekbote/virtualbrowser:new-build'
                }
            }

            stage('Docker-Image-Push') {
                steps {
                    script {
                        withDockerRegistry(credentialsId: 'docker-cred', toolName: 'Docker') {
                        sh 'docker push vamshiyekbote/virtualbrowser:new-build'
                        }
                    }
                }
            }
            
            stage('Deploy') {
                steps {
                    sh "docker-compose up -d "
                }
            }
    }
}
