pipeline {

    options {
        buildDiscarder(logRotator(numToKeepStr: "3"))
        disableConcurrentBuilds()
        timeout(time: 10, unit: "HOURS")
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
                    withCredentials([string(credentialsId: 'gpg-key-passphrase', variable: 'GPG_PASSPHRASE')]) {
                        configFileProvider([configFile(fileId: 'gpg-key', variable: 'GPG_KEY_FILE')]) {
                            sh '/setup-gpg.sh || true'
                        }
                    }
                }
            }
        }

        stage('Compile') {
            steps {
                container('maven') {
                    configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
                        withMaven() {
                            sh '$MVN_CMD -s $MAVEN_SETTINGS clean install'
                        }
                    }
                }
            }
        }

        stage('Release') {
    		when {
		        branch "develop"
			}
            steps {
                container('maven') {
					withCredentials([sshUserPrivateKey(credentialsId: 'jenkins', keyFileVariable: 'jenkins_id', passphraseVariable: 'jenkins_pass', usernameVariable: 'jenkins_user')]) {
                    	configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
                        	withMaven() {
                        		sh 'ssh-keyscan anrisoftware.com > /etc/ssh/ssh_known_hosts'
                        		sh 'cp ${jenkins_id} ${HOME}/.ssh/id_rsa'
                        		sh 'chmod go-rw ${HOME}/.ssh/id_rsa'
                            	sh '$MVN_CMD -s $MAVEN_SETTINGS -B -X release:prepare'
                            	sh '$MVN_CMD -s $MAVEN_SETTINGS -B -X release:perform'
                        	}
                        }
                    }
                }
            }
        } // stage

    }
}
