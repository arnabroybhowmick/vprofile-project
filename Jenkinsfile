pipeline {
    agent any

    tools {
        jdk "JDK-21"
        maven "Maven 3.9.9"
    }

    environment {
        SNAP_REPO      = 'Vprofile-snapshot'
        NEXUS_USER     = 'admin'
        NEXUS_PASS     = 'admin123'
        RELEASE_REPO   = 'vprofile-release'
        CENTRAL_REPO   = 'vprofile-central'
        NEXUSIP        =  "${env.NEXUSIP}"
        NEXUSPORT      = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN    = 'nexuslogin'
        SONARSERVER='sonarserver'
        SONARSCANNER='sonarscanner'
    }
     stages {

        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
        }

    }
    stage('SonarAnalysis') {
    environment {
        scannerHome = tool "${SONARSCANNER}"
    }

    steps {
        withSonarQubeEnv("${SONARSERVER}") {
            sh """
                ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=vprofile \
                -Dsonar.projectName=vprofile \
                -Dsonar.projectVersion=1.0 \
                -Dsonar.sources=src/ \
                -Dsonar.java.binaries=target/test-classes/com/visualpathit/account \
                -Dsonar.junit.reportsPath=target/surefire-reports/ \
                -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
            """
        }
    }
}