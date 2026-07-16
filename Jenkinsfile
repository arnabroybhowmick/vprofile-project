pipeline {
    agent any

    tools {
        jdk "JDK17"
        maven "MAVEN3.9"
    }

    environment {
        SNAP_REPO      = 'Vprofile-snapshot'
        NEXUS_USER     = 'admin'
        NEXUS_PASS     = 'admin123'
        RELEASE_REPO   = 'vprofile-release'
        CENTRAL_REPO   = 'vprofile-central'
        NEXUSIP        = '3.84.184.131'
        NEXUSPORT      = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN    = 'nexuslogin'
    }
     stages {

        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
        }

    }
}