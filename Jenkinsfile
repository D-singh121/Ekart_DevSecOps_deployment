pipeline {
    agent any
    
    tools { maven 'maven3'
            jdk 'jdk11'       
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/D-singh121/Ekart_DevSecOps_deployment.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
                
            }
        }
        
        stage('Unit Tests') {
            steps {
                echo 'Running the Test'
                sh " mvn test -DskipTests=true"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'sonar running'
                withSonarQubeEnv('sonar') {
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                   -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Deploy To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                   sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('Docker Build & tag Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName:'docker') {
                        sh "docker build -t devesh121/ekart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                sh "trivy image devesh121/ekart:latest > trivy-report.txt "
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName:'docker') {
                        sh "docker push devesh121/ekart:latest"
                    }
                }
            }
        }
        stage('Kubernetes Deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName:'', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess:false, serverUrl: 'https://172.31.8.162:6443') {
                        sh "kubectl apply -f deploymentservice.yml -n webapps"
                        sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}
