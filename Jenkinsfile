node {

    stage('Initialize') {
        checkout scm
    }

    stage('Build Code') {
         sh 'mvn clean install'
    }

    stage('Deploy Code') {
            nexusArtifactUploader( 
                credentialsId: 'sonatype-nexus-3', 
                groupId: 'com.eureka', 
                nexusUrl: '35.221.48.169:8081/nexus', 
                nexusVersion: 'nexus3', 
                protocol: 'http', 
                repository: 'maven-snapshots', 
                version: '1.0'
            )
    }

}
