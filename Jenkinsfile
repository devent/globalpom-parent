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

        stage('SonarQube Analysis') {
            steps {
                container('maven') {
					withSonarQubeEnv('sonarqube') {
	                    configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
	                        withMaven() {
	                            sh '$MVN_CMD -s $MAVEN_SETTINGS sonar:sonar'
	                        }
	                    }
	            	}
                }
            }
        }

        stage('Release') {
    		when {
		        branch "master"
			}
            steps {
                container('maven') {
                	configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
                        sh '/setup-ssh.sh'
                    	withMaven() {
                    	    sh 'git checkout develop'
                        	sh '$MVN_CMD -s $MAVEN_SETTINGS -B -X release:prepare'
                        	sh '$MVN_CMD -s $MAVEN_SETTINGS -B -X release:perform'
                    	}
                    }
                }
            }
        } // stage

    }
}
