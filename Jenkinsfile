node {

    stage('Initialize') {
        steps {
            checkout scm
        }
    }

    stage('Build Code') {
        steps {
            sh 'mvn clean install'
        }
    }

    stage('Deploy Code'){
        steps {
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

}