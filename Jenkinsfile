/bin/bash: line 1: qa: command not found   agent any
    
    environment {
        // DOCKER_REGISTRY = 'marbel89'
        DOCKER_CRED = credentials('DOCKERHUB_CONFIG')
        K3S_KUBECONFIG = credentials('KUBECONFIG')
    }
    
         stage('Build Docker Images') {
            steps {
                script {
                    // Build both services
                    docker.build("${DOCKER_REGISTRY}/movie-service:${BUILD_NUMBER}", "./movie-service")
                    docker.build("${DOCKER_REGISTRY}/cast-service:${BUILD_NUMBER}", "./cast-service")
                }
            }
        }
       
        stage('Push Docker Images') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        docker.image("${DOCKER_REGISTRY}/movie-service:${BUILD_NUMBER}").push()
                        docker.image("${DOCKER_REGISTRY}/cast-service:${BUILD_NUMBER}").push()
                    }
                }
            }
        }
