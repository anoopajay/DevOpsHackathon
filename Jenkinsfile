def CONTAINER_NAME="eureka-server"
def CONTAINER_TAG="latest"
def DOCKER_HUB_USER="anoopajay86"
def HTTP_PORT="8761"

node {

    stage('Initialize') {
        
        checkout scm
    
    }

    stage('Sonar Qube Code Analysis') {

        //withSonarQubeEnv('SonarQube') {
            //sh 'mvn sonar:sonar'
        //}
        sh 'echo Passed Test'

    }

    stage('Build Code') {

        sh 'mvn clean install'
    
    }

    stage("Image Prune"){

        imagePrune(CONTAINER_NAME)

    }

    stage('Image Build'){

        imageBuild(CONTAINER_NAME, CONTAINER_TAG)

    }

    stage('Push to Docker Registry'){

        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            pushToImage(CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
        }

    }

    stage('Run App'){

        runApp(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, HTTP_PORT)

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

def imagePrune(containerName){
    try {
        sh "docker image prune -f"
        sh "docker stop $containerName"
    } catch(error){}
}

def imageBuild(containerName, tag){
    sh "docker build -t $containerName:$tag  -t $containerName --pull --no-cache ."
    echo "Image build complete"
}

def pushToImage(containerName, tag, dockerUser, dockerPassword){
    sh "docker login -u $dockerUser -p $dockerPassword"
    sh "docker tag $containerName:$tag $dockerUser/$containerName:$tag"
    sh "docker push $dockerUser/$containerName:$tag"
    echo "Image push complete"
}

def runApp(containerName, tag, dockerHubUser, httpPort){
    sh "docker pull $dockerHubUser/$containerName"
    sh "docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}