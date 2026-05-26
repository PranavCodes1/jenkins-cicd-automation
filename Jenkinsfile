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

  - name: kubectl
    image: alpine/k8s:1.29.0
    command:
    - cat
    tty: true
'''
        }
    }

    environment {
        IMAGE = "pranavcodes1/jenkins-cicd-automation:${BUILD_NUMBER}"
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

        stage('Deploy to Kubernetes') {
            steps {

                container('kubectl') {

                    withCredentials([
                        file(
                            credentialsId: 'v4c-kubeconfig',
                            variable: 'KUBECONFIG'
                        )
                    ]) {

                       sh '''
kubectl version --client

kubectl get ns

kubectl create namespace $NAMESPACE \
--dry-run=client -o yaml | kubectl apply -f -

sed -i "s|__IMAGE_TAG__|'"$BUILD_NUMBER"'|g" k8s/deployment.yaml

kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
'''                    }

                }

            }
        }

    }

}
