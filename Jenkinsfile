pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH = '/opt/maven/bin:$PATH'
}
    stages {
        stage('Build') {
            steps {
               sh 'mvn clean deploy'
            }
        }
    }
}