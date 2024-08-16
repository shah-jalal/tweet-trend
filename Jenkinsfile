def registry = 'https://shahcloud.jfrog.io'
def imageName = 'shahcloud.jfrog.io/ttrend-docker-local/ttrend'
def version = '2.1.2'
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
                echo "---------------------- Build Started ----------------------"
               sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "---------------------- Build Completed ---------------------"
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
                echo '---------------------- Jar Publish Started ------------------------'
                def server = Artifactory.newServer url: registry+"/artifactory", credentialsId: 'artifactory_token'
                //def server = Artifactory.server 'jfrog-server'
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
                    echo '---------------------- Jar Publish Ended ------------------------'        
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    echo '---------------------- Docker Build Started ------------------------'
                    app = docker.build(imageName+":"+version)
                    echo '---------------------- Docker Build Completed -----------------------'
                }
            }
        }

        stage('Docker Publish') {
            steps {
                script {
                    echo '---------------------- Docker Publish Started ------------------------'
                    docker.withRegistry(registry, 'artifactory_token'){
                        app.push()
                    }
                    echo '---------------------- Docker Publish Completed -----------------------'
                }
            }
        }

    } //end stages
} //end pipeline
