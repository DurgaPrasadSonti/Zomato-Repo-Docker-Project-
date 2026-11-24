
pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'mysonar'
    }

    stages {
        stage('Clean WS') {
            steps {
                cleanWs()
            }
        }

        stage('Code Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/DurgaPrasadSonti/Zomato-Repo-Docker-Project-.git'
            }
        }

        stage('CQA') {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=zomato \
                        -Dsonar.projectKey=zomato
                    '''
                }
            }
        }

        stage('Quality Gates') {
            steps {
                script {
                    waitForQualityGate(
                        abortPipeline: false,
                        credentialsId: 'sonar-token'
                    )
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('OWASP') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Dockerfile') {
            steps {
                sh 'docker build -t durgaprasad19982001/sonti:zomato .'
            }
        }

        stage('trivy file') {
            steps {
                sh 'trivy fs . > trivy.txt'
            }
        }

        stage('ImageScan') {
            steps {
                sh 'trivy image durgaprasad19982001/sonti:zomato'
            }
        }

        stage('DockerPush') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh 'docker push durgaprasad19982001/sonti:zomato'
                    }
                }
            }
        }

        stage('Container') {
            steps {
                sh 'docker run -itd --name zomato -p 3000:3000 durgaprasad19982001/sonti:zomato'
            }
        }
    }
}
