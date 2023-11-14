pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    environment {
        PATH = ":/opt/apache-maven-3.9.5/bin:$PATH"
        registry = 'https://xtianechicajfrog.jfrog.io'
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
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/*.*",
                                "target": "libs-release-local/myapp/${env.BUILD_NUMBER}",
                                "props": "build.number=${env.BUILD_NUMBER};build.id=${env.BUILD_ID}",
                                "flat": "true"
                                "props" : "${properties}",
                                "exclusions": [ "*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload spec: uploadSpec
                    buildInfo.env.capture = true
                    server.publishBuildInfo buildInfo
                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }

    }
}