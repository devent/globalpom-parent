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
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: muellerpublic.de/anrisoftware-com
          operator: In
          values:
          - required
"""
        }
    }

    stages {

        stage("checkout") {
            steps {
                container('maven') {
                    checkout scm
                    sh "pwd"
                    sh "ls -al"
                }
            }
        }

        stage('Compile Code') {
            steps {
                container('maven') {
                    sh 'mvn compile'
                }
            }
        }

        stage('Test Code') {
            steps {
                container('maven') {
                    sh 'mvn test'
                }
            }
        }

    }
}
