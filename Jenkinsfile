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
    env:
    - name: HOME
      value: "/home/jenkins"
    - name: MAVEN_OPTS
      value: "-Xms450m -Xmx450G"
    resources:
      limits:
        cpu: 0.5
        memory: 500Mi
    tty: true
    securityContext:
      runAsUser: 10000
      fsGroup: 10000
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
                    sh "pwd"
                    sh "ls -al"
                    sh "echo \$HOME"
                    checkout scm
                    sh "ls -l /home/jenkins/workspace"
                }
            }
        }

        stage('Compile Code') {
            steps {
                container('maven') {
                    sh 'mvn compile'
                    sh "find / -name '.m2' -type d"
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
