pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: docker
                    image: docker:dind
                    securityContext:
                      privileged: true
                    volumeMounts:
                      - name: dind-storage
                        mountPath: /var/lib/docker
                  volumes:
                  - name: dind-storage
                    emptyDir: {}
            '''
        }
    }
   
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
                container('docker') {
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