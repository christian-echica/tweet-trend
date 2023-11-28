pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    environment {
        PATH = ":/opt/apache-maven-3.9.5/bin:$PATH"
        registry = 'https://xtianechicajfrog.jfrog.io/artifactory' 
    }

    stages {
        stage('build') {
            steps {
                echo "----------- build started ----------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "----------- build completed ----------"
            }
        }

        stage('test') {
            steps {
                echo "----------- unit test started ----------"
                sh 'mvn surefire-report:report'
                echo "----------- unit test Completed ----------"
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'xtianexica-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('xtianexica-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer(url: env.registry, credentialsId: 'artifact-cred')
                    def properties = "buildid=${env.BUILD_ID};commitid=${env.GIT_COMMIT}";
                    def buildInfo = Artifactory.newBuildInfo() 
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/com/valaxy/demo-workshop/2.1.2/*.*",
                                "target": "libs-release-local/com/valaxy/demo-workshop/2.1.2/",
                                "flat": "true", 
                                "recursive": "true", 
                                "props": "build.number=${env.BUILD_NUMBER};commit.id=${env.GIT_COMMIT}",
                                "exclusions": [ "*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    buildInfo.env.capture = true
                    buildInfo = server.upload spec: uploadSpec, buildInfo: buildInfo
                    server.publishBuildInfo buildInfo
                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    echo "----------- deployment started ----------"
                    sh './deploy.sh'
                    echo "----------- deployment completed ----------"
                }
            }
        }
    }
}