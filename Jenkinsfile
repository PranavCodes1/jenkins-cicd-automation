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

    tty: true
'''
        }
    }

    environment {
        IMAGE = 'pranavcodes1/jenkins-cicd-automation:latest'
        NAMESPACE = 'pranav-cicd'
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
                    sh '''
                    docker version
                    docker build -t $IMAGE .
                    '''
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
                    docker push $IMAGE
                    '''
                }
            }
        }

        stage('Verify Kubernetes Access') {
            steps {
                container('docker') {

                    sh '''
                    which kubectl
                    '''

                    sh '''
                    kubectl config current-context
                    '''

                    sh '''
                    kubectl get ns
                    '''
                }
            }
        }

        stage('Create Namespace') {
            steps {
                container('docker') {

                    sh '''
                    kubectl create namespace $NAMESPACE \
                    --dry-run=client -o yaml | kubectl apply -f -
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('docker') {

                    sh '''
                    kubectl apply -f k8s/deployment.yaml
                    '''

                    sh '''
                    kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }

    }

}
