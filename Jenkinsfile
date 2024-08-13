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
               sh 'mvn clean deploy'
            }
        }
    }
}