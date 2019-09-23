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
        nexusUrl: '35.245.247.242:8081', 
        nexusVersion: 'nexus3', 
        protocol: 'http', 
        repository: 'maven-releases', version: '1.0'
    
    }

    stage('Kubernetes Deployment') {


        sh 'gcloud container clusters get-credentials kube-cluster-1 --zone us-east1-b --project starlit-badge-253518'
        //sh 'kubectl create secret docker-registry regsecret --docker-server=10.150.0.4:8082 --docker-username=admin --docker-password=admin'

        sh 'kubectl delete deployment eureka-server || true'
        sh 'kubectl apply -f appdeploy.yaml'
        sh 'sleep 200'
        sh 'kubectl get services'

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