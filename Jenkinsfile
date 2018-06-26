pipeline {

    options {
        buildDiscarder(logRotator(numToKeepStr: "3"))
        disableConcurrentBuilds()
        timeout(time: 10, unit: "MINUTES")
    }

    agent {
        label 'maven-3-jdk-8'
    }

    stages {

        stage("Checkout") {
            steps {
                container('maven') {
                    checkout scm
                }
            }
        }

        stage('Setup') {
            steps {
                container('maven') {
                    configFileProvider([configFile(fileId: 'gpg-key', variable: 'GPG_KEY')]) {
                        sh 'cat $GPG_KEY'
                    }
                }
            }
        }

        stage('Compile Code') {
            steps {
                container('maven') {
                    configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
                        withMaven() {
                            sh '$MVN_CMD -s $MAVEN_SETTINGS clean package'
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                container('maven') {
                    withMaven() {
                        sh '$MVN_CMD deploy'
                    }
                }
            }
        }

    }
}
