    environment {
        // DOCKER_REGISTRY = 'marbel89'
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
                sh 'docker build -t test-build ./movie-service'
                sh 'docker build -t test-build ./cast-service'
            }
        }
        
        stage('Test Kubernetes Config') {
            steps {
                sh '''
                    export KUBECONFIG=$KUBECONFIG
                    kubectl get nodes
                '''
            }
        }
    }