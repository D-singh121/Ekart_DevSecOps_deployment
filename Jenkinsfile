pipeline {
    agent any
    
    tools { maven 'maven3.6'
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
                sh " mvn-test -DskipTests=true"
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
        
    }
}
