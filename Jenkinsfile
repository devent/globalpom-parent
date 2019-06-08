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
 * @version 1.2.0
 */
pipeline {

    options {
        buildDiscarder(logRotator(numToKeepStr: "3"))
        disableConcurrentBuilds()
        timeout(time: 60, unit: "MINUTES")
    }

    agent {
        label 'maven-3-jdk-12'
    }

    stages {

        /**
        * The stage will checkout the current branch.
        */
        stage("Checkout Build") {
            steps {
                container('maven') {
                    checkout scm
                }
            }
        } // stage

        /**
        * The stage will setup the container for the build.
        */
        stage('Setup Build') {
            steps {
                container('maven') {
                    withCredentials([string(credentialsId: 'gpg-key-passphrase', variable: 'GPG_PASSPHRASE')]) {
                        configFileProvider([configFile(fileId: 'gpg-key', variable: 'GPG_KEY_FILE')]) {
                            sh '/setup-gpg.sh'
                        }
                    }
                }
            }
        } // stage

        /**
        * The stage will compile and test on all branches.
        */
        stage('Compile and Test') {
            steps {
                container('maven') {
                    configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
                        withMaven() {
                            sh 'echo $MVN_CMD'
                            sh 'echo $MAVEN_SETTINGS'
                            sh '$MVN_CMD -s $MAVEN_SETTINGS clean install'
                        }
                    }
                }
            }
        } // stage

        /**
        * The stage will perform the SonarQube analysis on all branches.
        */
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
        } // stage

        /**
        * The stage will deploy the artifacts to the private repository.
        */
        stage('Deploy to Private') {
            steps {
                container('maven') {
                    configFileProvider([configFile(fileId: 'maven-settings-global', variable: 'MAVEN_SETTINGS')]) {
                        withMaven() {
                            sh '/setup-ssh.sh'
                            sh '$MVN_CMD -s $MAVEN_SETTINGS -B deploy'
                        }
                    }
                }
            }
        } // stage

        /**
        * The stage will perform a release from the develop branch.
        */
        stage('Release to Private') {
            when {
                branch 'develop'
                expression {
                    // skip stage if it is triggered by maven release.
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

        /**
        * The stage will deploy the artifacts and the generated site to the public repository from the master branch.
        */
        stage('Publish to Public') {
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
        
    } // stages

    post {
        success {
            timeout(time: 15, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }

    } // post
}
