pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t pranavcodes1/jenkins-cicd-automation:latest .'
            }
        }

    }

}
