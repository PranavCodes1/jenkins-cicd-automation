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
        IMAGE = "registry.vforeseetech.com/my-repository/jenkins-cicd-automation:${BUILD_NUMBER}"
        NAMESPACE = "pranav-cicd"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }
	
	stage('SonarQube Analysis') {
    steps {
        script {
            def scannerHome = tool 'SonarScanner'

            withSonarQubeEnv('SonarQube') {
                sh """
                ${scannerHome}/bin/sonar-scanner
                """
            }
        }
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
                            credentialsId: 'nexus-creds',
                            usernameVariable: 'NEXUS_USER',
                            passwordVariable: 'NEXUS_PASS'
                        )
                    ]) {

                        sh '''
                        echo "$NEXUS_PASS" | docker login registry.vforeseetech.com \
                        -u "$NEXUS_USER" \
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
                        ),
                        usernamePassword(
                            credentialsId: 'nexus-creds',
                            usernameVariable: 'NEXUS_USER',
                            passwordVariable: 'NEXUS_PASS'
                        )
                    ]) {

                        sh '''
                        kubectl version --client

                        kubectl create namespace $NAMESPACE \
                        --dry-run=client -o yaml | kubectl apply -f -

                        kubectl create secret docker-registry nexus-secret \
                          --docker-server=registry.vforeseetech.com \
                          --docker-username="$NEXUS_USER" \
                          --docker-password="$NEXUS_PASS" \
                          --namespace=$NAMESPACE \
                          --dry-run=client -o yaml | kubectl apply -f -

                        sed -i "s|__IMAGE_TAG__|$BUILD_NUMBER|g" k8s/deployment.yaml

                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml

                        kubectl rollout status deployment/flask-app -n $NAMESPACE --timeout=120s || true
                        '''
                    }
                }
            }
        }
    }
}
