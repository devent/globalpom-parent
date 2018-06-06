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
    - name: MAVEN_OPTS
        value: "-Xms450m -Xmx450G"
    volumeMounts:
    - mountPath: /root/.m2/repository
        name: jenkins
        subPath: m2-repository
    resources:
        limits:
        cpu: 0.5
        memory: 500Mi
    tty: true
    volumes:
    - name: jenkins
    persistentVolumeClaim:
        claimName: jenkins
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
