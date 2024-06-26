def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]
pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven'
    }
    environment {
        SCANNER_HOME = tool 'Sonar-Scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Pranay-OS18/Java-CICD-Project.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Scan') {
            steps {
                sh 'trivy fs --format table -o scan-report.html .'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonar-Server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=JVM-App -Dsonar.projectKey=JVM-App \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialId: 'Sonar-Token'
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Artifact to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'Global-Settings', jdk: 'JDK17', maven: 'Maven', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-Cred', url: '') {
                        sh 'docker build -t pranay18cr/jvm-image:v1 .'
                    }
                }
            }
        }
        stage('Image Scan') {
            steps {
                sh 'trivy image --format table -o scan-image-report.html pranay18cr/jvm-image:v1'
            }
        }
        stage('Push Image To Docker Hub') {
            steps {
                withDockerRegistry(credentialsId: 'Docker-Cred', url: '') {
                        sh 'docker push pranay18cr/jvm-image:v1'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'K8s-Cred', namespace: 'jvm-group', restrictKubeConfigAccess: false, serverUrl: 'https://10.0.1.49:6443') {
                    sh 'kubectl create -f deployment.yaml'
                }
            }
        }
        stage('Check Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'K8s-Cred', namespace: 'jvm-group', restrictKubeConfigAccess: false, serverUrl: 'https://10.0.1.49:6443') {
                    sh 'kubectl get pods -n jvm-group'
                }
            }
        }
    }
    post {
        always {
            echo 'Slack Notify'
            slackSend channel: '#jenkinscicd',
                 color: COLOR_MAP[currentBuild.currentResult],
                 message: "Build Successful"
        }
    }
}    
