node {

    stage('Initialize') {
        
        checkout scm
    
    }

    stage('Sonar Qube Code Analysis') {

        withSonarQubeEnv('sonarqube') {
            sh 'mvn sonar:sonar'
        }

    }

    stage('Build Code') {

         sh 'mvn clean install'
    
    }

    stage('Deploy Code') {

        nexusArtifactUploader( 
            artifacts: [[artifactId: 'nexus-artifact-uploader', classifier: 'debug', file: 'eureka-server-0.0.1-SNAPSHOT.jar', type: 'jar']], 
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
