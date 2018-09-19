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
		        branch 'develop'
		        expression {
					return !sh(script: "git --no-pager log -1 --pretty=%B", returnStdout: true).contains('[maven-release-plugin]')
				}
			}
            steps {
                container('maven') {
                	configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
                    	withMaven() {
	                        sh '/setup-ssh.sh'
                    	    sh 'git checkout develop && git pull origin develop'
                        	sh '$MVN_CMD -s $MAVEN_SETTINGS -B release:prepare'
                        	sh '$MVN_CMD -s $MAVEN_SETTINGS -B release:perform'
                    	}
                    }
                }
            }
        } // stage

        stage('Publish') {
    		when {
		        branch 'master'
			}
            steps {
                container('maven') {
                	configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
                    	withMaven() {
                            sh '$MVN_CMD -s $MAVEN_SETTINGS -Posssonatype -B deploy'
                    	}
                    }
                }
            }
        } // stage

    }
}
