node('docker-slave-general') { 
  def DockerImage = "webserver:v1.0"
  
  stage('Pre') { // Run pre-build steps
    cleanWs()
    sh "docker rm -f webserver || true"
  }
  
  stage('Git') { // Get code from GitLab repository
    git branch: 'master',
      url: 'https://github.com/hezil/flask-http.git'
  }
  
  stage('Build') { // Run the docker build
    sh "docker build --tag ${DockerImage} ."
  }
  
  stage('Run') { // Run the built image
    sh "docker run -d --name webserver --rm -p 8082:5000 ${DockerImage}; sleep 5"
  }
  
  stage('Test') { // Run tests on container
    def dockerOutput = sh (
        script: 'curl http://172.17.0.1:8082/goaway',
        returnStdout: true
        ).trim()
    sh "docker rm -f webserver"
    
    if ( dockerOutput == 'GO AWAY!' ) {
        currentBuild.result = 'SUCCESS'
    } else {
        currentBuild.result = 'FAILURE'
        sh "echo Webserver returned ${dockerOutput}"
    }
    return
  }
  
  stage('Push') { // Push the image to repository
   withDockerRegistry([ credentialsId: "hezil_dockerhub", url: "" ]) {
         sh "docker push ${DockerImage}"
       }
   sh "docker rmi ${DockerImage}"
   return
 }
}
