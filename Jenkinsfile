pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  hostNetwork: true
                  dnsPolicy: ClusterFirstWithHostNet
                  containers:
                  - name: docker
                    image: docker:dind
                    securityContext:
                      privileged: true
                    volumeMounts:
                      - name: dind-storage
                        mountPath: /var/lib/docker
                  - name: kubectl
                    image: dtzar/helm-kubectl:latest
                    command:
                      - cat
                    tty: true
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
        stage('Build') {
            steps {
                container('docker') {
                    script {
                        sh 'docker-compose build'
                    }
                }
            }
        }
        stage('Test') {
            steps {
                container('docker') {
                    script {
                        sh 'docker-compose up -d'
                        // Add tests here
                        sh 'docker-compose down'
                    }
                }
            }
        }
        stage('Push Docker Images') {
            steps {
                container('docker') {
                    sh 'docker images'
                    /*sh """
                    echo $DOCKER_CRED_PSW | docker login -u $DOCKER_CRED_USR --password-stdin
                    docker tag microservices-pipeline-movie_service ${DOCKER_REGISTRY}/movie-service:${BUILD_NUMBER}
                    docker tag microservices-pipeline-cast_service ${DOCKER_REGISTRY}/cast-service:${BUILD_NUMBER}
                    docker push ${DOCKER_REGISTRY}/movie-service:${BUILD_NUMBER}
                    docker push ${DOCKER_REGISTRY}/cast-service:${BUILD_NUMBER}
                    """*/
                }
            }
        }
        stage('Deploy to Dev') {
    steps {
        container('kubectl') {
                    withKubeConfig([
                        credentialsId: 'KUBECONFIG',
                        serverUrl: 'https://172.17.215.95:6443'
                    ]) {
                        sh '''
                            # Debug information
                            echo "=== Kubeconfig Debug ==="
                            kubectl config view
                            echo "\nCluster info:"
                            kubectl cluster-info
                            echo "\nNodes:"
                            kubectl get nodes
                            
                            # Create namespace if it doesn't exist
                            kubectl create namespace dev --dry-run=client -o yaml | kubectl apply -f -
                        '''
                        
                        sh """
                            helm upgrade --install microservices ./charts \
                                --set image.tag=${BUILD_NUMBER} \
                                --namespace dev
                        """
           
            }
        }
    }
}
    }
}