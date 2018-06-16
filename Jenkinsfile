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

        stage("checkout") {
            steps {
                container('maven') {
                    checkout scm
                }
            }
        }

        stage('Compile Code') {
            steps {
                container('maven') {
                    withMaven() {
                        sh '$MVN_CMD compile'
                    }
                }
            }
        }

        stage('Test Code') {
            steps {
                container('maven') {
                    withMaven() {
                        sh '$MVN_CMD test'
                    }
                }
            }
        }

    }
}
