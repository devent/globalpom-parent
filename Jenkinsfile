pipeline {

    options {
        buildDiscarder(logRotator(numToKeepStr: "3"))
        disableConcurrentBuilds()
        timeout(time: 10, unit: "MINUTES")
    }

    agent {
        kubernetes {
            label 'mypod'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: some-label-value
spec:
  containers:
  - name: maven
    image: maven:alpine
    command:
    - cat
    tty: true
  - name: busybox
    image: busybox
    command:
    - cat
    tty: true
"""
        }
    }

    stages {
        stage("checkout") {
            checkout scm
        }
        stage('Run maven') {
            steps {
                container('maven') {
                    sh 'mvn -version'
                    sh 'mount'
                }
                container('busybox') {
                    sh '/bin/busybox'
                }
            }
        }
    }
}
