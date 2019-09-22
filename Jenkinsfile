node {

    stage('Initialize') {
        
        checkout scm
    
    }

    stage('Sonar Qube Code Analysis') {

        withSonarQubeEnv('SonarQube') {
            sh 'mvn sonar:sonar'
        }

    }

    stage('Build Code') {

         sh 'mvn clean install'
    
    }

    stage('Deploy Code') {

        nexusArtifactUploader artifacts: [[artifactId: 'nexus-artifact-uploader', classifier: 'debug', file: 'eureka-server/target/eureka-server-0.0.1-SNAPSHOT.jar', type: 'jar'], 
        [artifactId: 'nexus-artifact-uploader', classifier: 'debug', file: 'pom.xml', type: 'xml'], 
        [artifactId: 'nexus-artifact-uploader', classifier: 'debug', file: 'eureka-server/target/eureka-server-0.0.1-SNAPSHOT.jar', type: 'jar']], 
        credentialsId: 'sonatype-nexus-3', 
        groupId: 'com.eureka', 
        nexusUrl: '35.221.48.169:8081/nexus', 
        nexusVersion: 'nexus3', 
        protocol: 'http', 
        repository: 'maven-releases', version: '1.0'
    
    }

}
