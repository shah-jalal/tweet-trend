pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH+EXTRA = '/opt/maven/bin:$PATH'
}
    stages {
        stage('Build') {
            steps {
               sh 'mvn clean deploy'
            }
        }
    }
}