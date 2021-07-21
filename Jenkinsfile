// Uses Declarative syntax to run commands inside a container.
pipeline {
  agent none

  stages {
    stage('Build Docker Image') {
      agent {
        kubernetes {

          yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:19.03.1-dind
    securityContext:
      privileged: true
    env:
      - name: DOCKER_TLS_CERTDIR
        value: ""
'''
         
          // Can also wrap individual steps:
          // container('shell') {
          //     sh 'hostname'
          // }
          defaultContainer 'docker'
        }
      }
      stages {
        stage('Checkout Source') {
          steps {
            git 'https://github.com/nashad-abdul-rahiman/hellowhale.git'
          }
        }
        stage("Build image") {
          steps {
            sh 'docker build -t nashadabdulrahiman/hellowhale:${BUILD_ID} .'
            sh 'docker tag nashadabdulrahiman/hellowhale:${BUILD_ID} nashadabdulrahiman/hellowhale:latest'
          }
        }
        stage("Push image") {
          steps {
            withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
              sh 'docker login -u $USERNAME -p $PASSWORD'
            }
            sh 'docker push nashadabdulrahiman/hellowhale:${BUILD_ID}'
            sh 'docker push nashadabdulrahiman/hellowhale:latest'
          }
        }
      }

    }

    
stage("Deploy") {
      agent {
        kubernetes {

          yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: 'alpine/k8s:1.20.4'
    securityContext:
      privileged: true
    command:
     - cat
    tty: true
'''
         
          // Can also wrap individual steps:
          // container('shell') {
          //     sh 'hostname'
          // }
          defaultContainer 'docker'
        }
      }
      steps {
        git 'https://github.com/nashad-abdul-rahiman/hellowhale.git'
        withCredentials([file(credentialsId: 'k3', variable: 'KUBECRED')]) {
            sh 'mkdir -pv /root/.kube/'
            sh 'cat $KUBECRED > ~/.kube/config'
        }
        
        sh 'kubectl apply -f hellowhale.yml'
      
      }
          
    }
  }

}
