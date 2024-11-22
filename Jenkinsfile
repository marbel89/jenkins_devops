pipeline {
    agent any
   
    environment {
        DOCKER_REGISTRY = 'marbel89'
        DOCKER_CRED = credentials('DOCKERHUB_CONFIG')
        K3S_KUBECONFIG = credentials('KUBECONFIG')
    }
   
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
       
        stage('Test Docker Build') {
            steps {
                script {
                    // Build movie service
                    dir('movie-service') {
                        sh 'docker build -t test-build-movie .'
                    }
                    // Build cast service
                    dir('cast-service') {
                        sh 'docker build -t test-build-cast .'
                    }
                }
            }
        }
       
        stage('Test Kubernetes Config') {
            steps {
                sh '''
                    export KUBECONFIG=$K3S_KUBECONFIG
                    kubectl get nodes
                '''
            }
        }
    }
}