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
                  - name: kubectl
                    image: bitnami/kubectl
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
                    withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_CONFIG', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh """
                            echo \$PASSWORD | docker login -u \$USERNAME --password-stdin
                            docker tag jenkins_devops_exams_movie_service ${DOCKER_REGISTRY}/movie-service:${BUILD_NUMBER}
                            docker tag jenkins_devops_exams_cast_service ${DOCKER_REGISTRY}/cast-service:${BUILD_NUMBER}
                            docker push ${DOCKER_REGISTRY}/movie-service:${BUILD_NUMBER}
                            docker push ${DOCKER_REGISTRY}/cast-service:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }
        stage('Deploy to Dev') {
            when { branch 'develop' }
            steps {
                container('kubectl') {
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