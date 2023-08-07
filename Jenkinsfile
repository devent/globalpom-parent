/**
 * Copyright 2011-2023 Erwin Müller <erwin.mueller@anrisoftware.com>
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
 * @since 4.6.2
 * @version 1.4.1
 */
def groupId
def artifactId
def version
def isSnapshot

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
        } // stage

        /**
        * Setups the pipeline.
        */
        stage("Setup") {
            steps {
                container("maven") {
                    script {
                        groupId = sh script: 'mvn -s /m2/settings.xml help:evaluate -Dexpression=project.groupId -q -DforceStdout', returnStdout: true
                        artifactId = sh script: 'mvn -s /m2/settings.xml help:evaluate -Dexpression=project.artifactId -q -DforceStdout', returnStdout: true
                        version = sh script: 'mvn -s /m2/settings.xml help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
                        isSnapshot = (version =~ /(?i).*-snapshot$/).matches()
                        echo "${groupId}/${artifactId}:${version} snapshot: ${isSnapshot}"
                    }
                }
            }
        } // stage

        /**
        * The stage will compile, test and deploy on all branches.
        */
        stage("Compile, Test and Deploy") {
            steps {
                container("maven") {
                    script {
                        if (isSnapshot) {
                            sh "/setup-gpg.sh; mvn -s /m2/settings.xml -B clean install site:site deploy site:deploy"
                        } else {
                            sh "/setup-gpg.sh; mvn -s /m2/settings.xml -B clean install site:site site:deploy"
                        }
                    }
                }
            }
        } // stage

        /**
        * The stage will deploy the artifacts and the generated site to the public repository from the main branch.
        */
        stage("Publish to Private") {
            when {
                allOf {
                    expression { !isSnapshot }
                    branch "main"
                }
            }
            steps {
                container("maven") {
                    sh "/setup-gpg.sh; mvn -s /m2/settings.xml -B deploy"
                }
            }
        } // stage

        /**
        * The stage will deploy the artifacts and the generated site to the public repository from the main branch.
        */
        stage("Publish to Public") {
            when {
                allOf {
                    expression { !isSnapshot }
                    branch "main"
                }
            }
            steps {
                container("maven") {
                    sh "/setup-gpg.sh; mvn -s /m2/settings.xml -Posssonatype -B deploy"
                }
            }
        } // stage

    } // stages

    post {
        success {
            container("maven") {
                script {
                    manager.createSummary("document.png").appendText("<a href=\"${env.JAVADOC_URL}/${groupId}/${artifactId}/${version}/index.html\">View Maven Site</a>", false)
                }
            }
        }
    } // post
        
}
