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
                    withCredentials([string(credentialsId: 'gpg-key-passphrase', variable: 'PASSPHRASE')]) {
                    configFileProvider([configFile(fileId: 'gpg-key', variable: 'GPG_KEY')]) {
                        sh '''
                            mkdir ~/.gnupg
                            chmod 700 ~/.gnupg
                            echo "use-agent" >> ~/.gnupg/gpg.conf
                            echo "pinentry-mode loopback" >> ~/.gnupg/gpg.conf
                            echo "allow-loopback-pinentry" >> ~/.gnupg/gpg-agent.conf
                            echo RELOADAGENT | gpg-connect-agent
                            echo "$PASSPHRASE" | base64 -d | gpg --passphrase-fd 0 --allow-secret-key-import --import $GPG_KEY
                        '''
                    }
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
                    configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
                        withMaven() {
                            sh '$MVN_CMD deploy'
                        }
                    }
                }
            }
        }

    }
}
