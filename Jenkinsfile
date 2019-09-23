def CONTAINER_NAME="eureka-server"
def CONTAINER_TAG="latest"
def HTTP_PORT="8761"
def NEXUS3_IP="35.245.247.242"
def NEXUS3_PORT="8082"

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

    stage("Image Prune"){

        imagePrune(CONTAINER_NAME)

    }

    stage('Image Build'){

        imageBuild(CONTAINER_NAME, CONTAINER_TAG)

    }

    stage('Push to Nexus Registry'){

        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            pushToImage(CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
        }

    }

    stage('Run Docker Image'){

        runApp(CONTAINER_NAME, CONTAINER_TAG, HTTP_PORT)

    }

    stage('Copy Artifacts to Sonatype Nexus 3') {

        nexusArtifactUploader artifacts: [[artifactId: 'nexus-artifact-uploader', classifier: 'debug', file: 'eureka-server/target/eureka-server-0.0.1-SNAPSHOT.jar', type: 'jar'], 
        [artifactId: 'nexus-artifact-uploader', classifier: 'debug', file: 'eureka-server/target/eureka-server-0.0.1-SNAPSHOT.jar', type: 'jar']], 
        credentialsId: 'sonatype-nexus-3', 
        groupId: 'com.eureka', 
        nexusUrl: '35.245.247.242:8081/nexus', 
        nexusVersion: 'nexus3', 
        protocol: 'http', 
        repository: 'maven-releases', version: '1.0'
    
    }

    stage('Kubernetes Deployment') {

        sh 'kubectl run eureka-server --image=35.245.247.242:8082/eureka-server:latest --port=8761'

    }

}

def imagePrune(containerName){
    try {
        sh "docker image prune -f"
        sh "docker container stop $containerName"
    } catch(error){}
}

def imageBuild(containerName, tag){
    sh "docker build -t $containerName:$tag  -t $containerName --pull --no-cache ."
    echo "Image build complete"
}

def pushToImage(containerName, tag, dockerUser, dockerPassword){
    sh "docker login 35.245.247.242:8082 -u $dockerUser -p $dockerPassword"
    sh "docker tag $containerName:$tag 35.245.247.242:8082/$containerName:$tag"
    sh "docker push 35.245.247.242:8082/$containerName:$tag"
    echo "Image push complete"
}

def runApp(containerName, tag, httpPort){
    sh "docker run -d --rm -p $httpPort:$httpPort $containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}