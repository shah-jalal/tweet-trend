pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    options {
        timeout(time: 1, unit: 'HOURS')
    }
environment {
    PATH = "/opt/maven/bin:${env.PATH}"
}
    stages {
        stage('Build') {
            steps {
                echo "------------- build started --------------"
               sh 'mvn compile'
                echo "------------- build completed --------------"
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

        stage('Quality Gate') {
            steps{
                script{
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }

                }
            }
            
        }
    }
}