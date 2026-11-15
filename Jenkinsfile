
pipeline {

    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven 3.9'
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "54.227.231.252:8081"
        NEXUS_REPOSITORY = "maven-releases"
        NEXUS_REPO_ID = "maven-releases"
        NEXUS_CREDENTIAL_ID = "admin"
        ARTVERSION = "${env.BUILD_ID}"
    }

    stages {

        // 1. BUILD
        stage('BUILD') {
            steps {
                sh 'mvn clean install -DskipTests'
            }

            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        // 2. UNIT TEST
        stage('UNIT TEST') {
            steps {
                sh 'mvn test'
            }
        }

        // 3. INTEGRATION TEST
        stage('INTEGRATION TEST') {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        // 4. CHECKSTYLE
        stage('CODE ANALYSIS WITH CHECKSTYLE') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }

            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        // 5. SONARQUBE CLOUD ANALYSIS
        stage('CODE ANALYSIS WITH SONARQUBE') {

            environment {
                scannerHome = tool 'sonarscanner4'
            }

            steps {

                withSonarQubeEnv('sonar-pro') {

                    withCredentials([
                        string(
                            credentialsId: 'sonar_Token',
                            variable: 'SONAR_TOKEN'
                        )
                    ]) {

                        sh '''
                            "${scannerHome}/bin/sonar-scanner" \
                            -Dsonar.organization=arnabroybhowmick \
                            -Dsonar.projectKey=arnabroybhowmick_vprofile-project \
                            -Dsonar.projectName=vprofile-repo \
                            -Dsonar.projectVersion=1.0 \
                            -Dsonar.sources=src/ \
                            -Dsonar.token="$SONAR_TOKEN" \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.junit.reportPaths=target/surefire-reports/ \
                            -Dsonar.jacoco.reportPaths=target/jacoco.exec \
                            -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                        '''
                    }
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // 6. PUBLISH TO NEXUS
        stage('Publish to Nexus Repository Manager') {

            steps {
                script {

                    def pom = readMavenPom file: 'pom.xml'

                    def packaging = pom.packaging ?: 'jar'

                    def filesByGlob = findFiles(
                        glob: "target/*.${packaging}"
                    )

                    if (filesByGlob.length == 0) {
                        error "No ${packaging} artifact found in target/"
                    }

                    def artifactPath = filesByGlob[0].path

                    echo "Publishing artifact: ${artifactPath}"

                    nexusArtifactUploader(

                        nexusVersion: env.NEXUS_VERSION,
                        protocol: env.NEXUS_PROTOCOL,
                        nexusUrl: env.NEXUS_URL,

                        groupId: pom.groupId,
                        version: env.ARTVERSION,
                        repository: env.NEXUS_REPOSITORY,
                        credentialsId: env.NEXUS_CREDENTIAL_ID,

                        artifacts: [

                            [
                                artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: packaging
                            ],

                            [
                                artifactId: pom.artifactId,
                                classifier: '',
                                file: 'pom.xml',
                                type: 'pom'
                            ]
                        ]
                    )
                }
            }
        }

        // 7. ANSIBLE DEPLOYMENT
        stage('Ansible Deploy to staging') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'staging-app-login',
                        usernameVariable: 'STAGING_USER',
                        passwordVariable: 'STAGING_PASS'
                    )
                ]) {

                    ansiblePlaybook(

                        inventory: 'ansible/stage.inventory',
                        playbook: 'ansible/site.yml',
                        installation: 'ansible',
                        colorized: true,

                        credentialsId: 'appLogin',
                        disableHostKeyChecking: true,

                        extraVars: [

                            USER: env.STAGING_USER,
                            PASS: env.STAGING_PASS,

                            nexusip: '172.31.5.4',
                            reponame: 'maven-releases',
                            groupid: 'QA',

                            time: env.BUILD_TIMESTAMP ?: '',
                            build: env.BUILD_ID,

                            artifactid: 'vproapp',

                            vprofile_version:
                                "vproapp-${env.BUILD_ID}-${env.BUILD_TIMESTAMP ?: env.BUILD_ID}.war"
                        ]
                    )
                }
            }
        }
    }
}
