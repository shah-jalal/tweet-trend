pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH = "/opt/maven/bin:${env.PATH}"
}
    stages {
        stage('Build') {
            steps {
               sh 'mvn compile'
            }
        }

        stage('SonarQube analysis') {
            environment { 
                scannerHome = tool 'sonar-scanner'
            }
            steps{
                withSonarQubeEnv('sonar-server') { 
                sh "${scannerHome}/bin/sonar-scanner"
                }
            }    
        }
    }
}