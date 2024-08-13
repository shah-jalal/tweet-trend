pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    stages {
        stage('Build') {
            steps {
                'mvn clean deploy'
            }
        }
    }
}