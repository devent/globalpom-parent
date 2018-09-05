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
                        	sh "rm doc*"
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
                	sshagent (credentials: ['jenkins']) {
                    	configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
                        	withMaven() {
                        		sh 'ssh-keygen > /etc/ssh/ssh_known_hosts'
                        		sleep time: 1, unit: "HOURS"
                            	sh '$MVN_CMD -s $MAVEN_SETTINGS -B release:prepare'
                            	sh '$MVN_CMD -s $MAVEN_SETTINGS -B release:perform'
                        	}
                    	}
                    }
                }
            }
        }

    }
}
