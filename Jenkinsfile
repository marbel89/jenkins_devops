pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'marbel89'
        DOCKER_CRED = credentials('dockerhub-credentials')
        K3S_KUBECONFIG = credentials('k3s-config')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/fastapiapp:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        docker.image("${DOCKER_REGISTRY}/fastapiapp:${BUILD_NUMBER}").push()
                    }
                }
            }
        }
        
        stage('Deploy to Dev') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    sh """
                        export KUBECONFIG=\$K3S_KUBECONFIG
                        helm upgrade --install fastapiapp ./charts \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30007 \
                            --namespace dev
                    """
                }
            }
        }
        
        stage('Deploy to QA') {
            when {
                branch 'qa'
            }
            steps {
                script {
                    sh """
                        export KUBECONFIG=\$K3S_KUBECONFIG
                        helm upgrade --install fastapiapp ./charts \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30008 \
                            --namespace qa
                    """
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'staging'
            }
            steps {
                script {
                    sh """
                        export KUBECONFIG=\$K3S_KUBECONFIG
                        helm upgrade --install fastapiapp ./charts \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30009 \
                            --namespace staging
                    """
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            input {
                message "Deploy to production?"
                ok "Yes"
            }
            steps {
                script {
                    sh """
                        export KUBECONFIG=\$K3S_KUBECONFIG
                        helm upgrade --install fastapiapp ./charts \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30010 \
                            --namespace prod
                    """
                }
            }
        }
    }
    
    post {
        always {
            sh 'docker logout'
            cleanWs()
        }
    }
}