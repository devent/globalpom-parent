/**
 * Copyright © 2011 Erwin Müller (erwin.mueller@anrisoftware.com)
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
/**
 * Builds and deploys the project.
 *
 * @author Erwin Mueller, erwin.mueller@deventm.org
 * @since 4.5.1
 * @version 1.3.0
 */
pipeline {

    options {
        buildDiscarder(logRotator(numToKeepStr: "3"))
        disableConcurrentBuilds()
        timeout(time: 60, unit: "MINUTES")
    }

    agent {
        label "maven"
    }

    stages {

        /**
        * The stage will checkout the current branch.
        */
        stage("Checkout Build") {
            steps {
                container("maven") {
                    checkout scm
                }
            }
        }

        /**
        * The stage will compile, test and deploy on all branches.
        */
        stage("Compile, Test and Deploy") {
            when {
                allOf {
                    not { branch "mainx" }
                }
            }
            steps {
                container("maven") {
                    sh "/setup-gpg.sh; /setup-ssh.sh; mvn -s /m2/settings.xml -X -B clean install site:site deploy"
                }
            }
        }

		/**
		* The stage will deploy the generated site for feature branches.
		*/
        stage("Deploy Site") {
    		when {
    			allOf {
					not { branch "master" }
					not { branch "develop" }
				}
			}
            steps {
                container("maven") {
                	configFileProvider([configFile(fileId: "MAVEN_SETTINGS", variable: "MAVEN_SETTINGS")]) {
                    	withMaven() {
	                        sh "/setup-ssh.sh"
                        	sh "$MVN_CMD -s $MAVEN_SETTINGS -B site:deploy"
                    	}
                    }
                }
            }
        } // stage

		/**
		* The stage will perform a release from the develop branch.
		*/
        stage("Release to Private") {
    		when {
		        branch "develop"
		        expression {
		        	// skip stage if it is triggered by maven release.
					return !sh(script: "git --no-pager log -1 --pretty=%B", returnStdout: true).contains("[maven-release-plugin]")
				}
			}
            steps {
	            timeout(time: 15, unit: "MINUTES") {
	                waitForQualityGate abortPipeline: true
	            }
                container("maven") {
                	configFileProvider([configFile(fileId: "MAVEN_SETTINGS", variable: "MAVEN_SETTINGS")]) {
                    	withMaven() {
	                        sh "/setup-ssh.sh"
                    	    sh "git checkout develop && git pull origin develop"
                        	sh "$MVN_CMD -s $MAVEN_SETTINGS -B release:prepare"
                        	sh "$MVN_CMD -s $MAVEN_SETTINGS -B release:perform"
                    	}
                    }
                }
            }
        } // stage

		/**
		* The stage will deploy the artifacts and the generated site to the public repository from the master branch.
		*/
        stage("Publish to Public") {
    		when {
		        branch "master"
			}
            steps {
                container("maven") {
                	configFileProvider([configFile(fileId: "MAVEN_SETTINGS", variable: "MAVEN_SETTINGS")]) {
                    	withMaven() {
                            sh "$MVN_CMD -s $MAVEN_SETTINGS -Posssonatype -B deploy"
                    	}
                    }
                }
            }
        } // stage

    } // stages

    post {
        success {
           script {
               pom = readMavenPom file: "pom.xml"
               manager.createSummary("document.png").appendText("<a href=\"${env.JAVADOC_URL}/${pom.groupId}/${pom.artifactId}/${pom.version}/\">View Maven Site</a>", false)
            }
        }
    } // post
        
}
