def appname = "hello-newapp"
def repo = "lizaaliza"  // Replace with your DockerHub username
def appimage = "${repo}/${appname}"
def apptag = "${env.BUILD_NUMBER}"

podTemplate(containers: [
      containerTemplate(name: 'jnlp', image: 'jenkins/inbound-agent', ttyEnabled: true),
      containerTemplate(
        name: 'docker', 
        image: 'docker:dind', 
        command: 'cat', 
        ttyEnabled: true, 
        privileged: true),
        args: '--storage-driver=vfs --host=tcp://0.0.0.0:2375'
  ],
   volumes: [
    emptyDirVolume(mountPath: '/var/run', memory: false) 
  ]) 
  {
    node(POD_LABEL) {
        stage('chackout') {
            container('jnlp') {
            sh '/usr/bin/git config --global http.sslVerify false'
            checkout scm
          }
        } // end chackout

        stage('build') {
            container('docker') {
              echo "Building docker image..."
              sh "echo docker push $appimage"
              sh "docker build -t $appimage:$apptag ."
            }
        } //end build

        stage('Push'){
            withCredentials([usernamePassword(credentialsId: 'hello-newapp', usernameVariable: 'repo', passwordVariable: 'PASS')]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push $appimage:$apptag
			docker push $appimage:latest
                    '''
                }
        }
    }
}
