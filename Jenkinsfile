def registry = 'https://shahcloud.jfrog.io'
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
                echo "------------- Build Started --------------"
               sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "------------- Build Completed --------------"
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
                        def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }

                }
            }
            
        }
        
        stage('Jar Published') {
        steps {
            script {
                echo '----------------------Jar Publish Started------------------------'
                def server = Artifactory.newServer url: registry+"/artifactory", credentialsId: 'jfrog-cred'
                def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "jar-libs-release-local/{1}",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
    
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(spec: uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '----------------------Jar Publish Ended------------------------'        
                }
            }
        }

    } //end stages
} //end pipeline
