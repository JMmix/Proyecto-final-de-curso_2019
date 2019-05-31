/*Declarative Pipeline*/
 pipeline {
  agent {
    kubernetes {
      label 'def'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: some-label-value
spec:
  containers:
  - name: docker
    image: docker:dind
    securityContext:
      privileged: true
    command:
    - cat
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
      subPath: docker.sock
    tty: true
  - name: helm
    image: alpine/helm:2.13.1
    command:
    - cat
    tty: true
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/
"""
    }
  }
    options {
        skipDefaultCheckout true
    }
  stages {
    stage('Building Container') {
      steps {
        container('docker') {
                        sh 'apk update'
                        sh 'apk add curl'
                        sh 'curl -O https://raw.githubusercontent.com/JMmix/jenkins-webhook/master/Dockerfile'
                        withCredentials([usernamePassword(credentialsId: 'gitlab-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]){
                            script {
                                docker.withRegistry('https://registry.gitlab.com', 'gitlab-cred') { 
                                app = docker.build("registry.gitlab.com/jmmix/automated.website:${env.BUILD_ID}")
                                app.push()
                                app.push('latest')
                            }
                        }
                    }
                }
            }
        }
    stage('Deploying to Kubernetes') {
        steps {
        container('helm') {
            sh 'helm init --client-only'
                sh 'helm fetch https://github.com/JMmix/jenkins-webhook/raw/master/src/test-0.1.0.tgz'
                sh '''
                helm upgrade apache test-0.1.0.tgz -i --recreate-pods
                '''
                }
            }
        }
    }
}
/* Scripted Pipeline*/
/*
node {
    stage('Building Docker image') {
        sh 'curl -O https://raw.githubusercontent.com/JMmix/jenkins-webhook/master/Dockerfile'
        withCredentials([usernamePassword(credentialsId: 'gitlab-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]){
            sh 'docker login registry.gitlab.com -u $DOCKER_USER -p $DOCKER_PASSWORD'
            app = docker.build("registry.gitlab.com/jmmix/automated.website:${env.BUILD_ID}")
               app.push()
               app.push('latest')
            }
        }
}
def label = "mypod-${UUID.randomUUID().toString()}"
        podTemplate(label: label, containers: [
        containerTemplate(name: 'helm', image: 'alpine/helm', ttyEnabled: true, command: 'cat')
        ]){
    node(label){
        stage('Deploying to Kubernetes') {
            container('helm'){
                sh 'helm init --client-only'
                sh 'helm fetch https://github.com/JMmix/jenkins-webhook/raw/master/test-0.1.0.tgz'
                sh '''
                helm upgrade apache test-0.1.0.tgz -i --recreate-pods
                '''
                }
            }
        }
    }
    */