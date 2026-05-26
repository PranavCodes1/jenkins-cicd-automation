pipeline {

    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:

  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
    command:
    - dockerd
    args:
    - --host=unix:///var/run/docker.sock

  - name: jnlp
'''
        }
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                container('docker') {
                    sh 'docker version'
                    sh 'docker build -t pranavcodes1/jenkins-cicd-automation:latest .'
                }
            }
        }

	stage('Docker Login') {
    steps {
        container('docker') {
            withCredentials([
                usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                sh '''
                echo "$DOCKER_PASS" | docker login \
                -u "$DOCKER_USER" \
                --password-stdin
                '''
            }
        }
    }
}

stage('Push Image') {
    steps {
        container('docker') {
            sh '''
            docker push pranavcodes1/jenkins-cicd-automation:latest
            '''
        }
    }
}

    }
}
