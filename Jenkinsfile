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
                            sh '/setup-gpg.sh'
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
					withCredentials([sshUserPrivateKey(credentialsId: 'jenkins', keyFileVariable: 'SSH_FILE', passphraseVariable: 'SSH_USER', usernameVariable: 'SSH_PASS')]) {
                    	configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
                        	withMaven() {
                        	    sh '/setup-ssh.sh'
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
