def appname = "hello-newapp"
def repo = "lizaaliza" // Replace with your DockerHub username
def appimage = "${repo}/${appname}"
def apptag = "${env.BUILD_NUMBER}"

podTemplate(
    containers: [
        containerTemplate(
            name: 'jnlp',
            image: 'jenkins/inbound-agent',
            ttyEnabled: true
        ),
        containerTemplate(
            name: 'docker',
            image: 'docker:dind',
            command: 'dockerd',
            ttyEnabled: true,
            privileged: true,
            args: '--storage-driver=vfs --host=tcp://0.0.0.0:2375'
        ),
        containerTemplate(
            name: 'helm',
            image: 'alpine/helm:latest',
            command: 'cat',
            ttyEnabled: true
        ),
    ],
    volumes: [
        emptyDirVolume(mountPath: '/var/run', memory: false)
    ]
) {
    node(POD_LABEL) {
        stage('checkout') {
            container('jnlp') {
                sh '/usr/bin/git config --global http.sslVerify false'
                checkout scm
            }
        } // end checkout

        stage('build') {
            container('docker') {
                withEnv(['DOCKER_HOST=tcp://localhost:2375']) {
                    echo "Building docker image..."
                    sh "echo docker push ${appimage}"
                    sh "docker build -t ${appimage}:${apptag} -t ${appimage}:latest ."
                }
            }
        } // end build

        stage('Push') {
            container('docker') {
                withEnv(['DOCKER_HOST=tcp://localhost:2375']) {
                    withCredentials([usernamePassword(credentialsId: 'hello-newapp', usernameVariable: 'repo', passwordVariable: 'PASS')]) {
                        sh """
                            echo "$PASS" | docker login -u "$repo" --password-stdin
                            docker push ${appimage}:${apptag}
                            docker push ${appimage}:latest
                        """
                    }
                }
            }
        } 

        stage('Deploy'){
           sh " echo helm template hello-newapp ./chart > hello-newapp.yaml"
        }
    } 
} 
