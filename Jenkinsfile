pipeline {
    agent any

    environment {
        IMAGE_NAME = "todoappproject"
        DOCKER_HUB_REPO = "achrafbrini007"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning the repository..."
                git branch: 'main', url: 'https://github.com/Kazu81/ToDoAppProject.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKER_HUB_REPO}/${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing the image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USR', passwordVariable: 'DOCKERHUB_PSW')]) {
                    sh '''
                      echo "$DOCKERHUB_PSW" | docker login --username "$DOCKERHUB_USR" --password-stdin
                      docker push ${DOCKER_HUB_REPO}/${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    script {
                        sh '''
                          export KUBECONFIG=$KUBECONFIG
                          kubectl apply -f react-dpl.yml --validate=false
                          kubectl rollout status deployment/todoappproject-deployment
                        '''
                    }
                }
            }
        }

        stage("Update Kubernetes Deployment") {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    script {
                        sh '''
                          export KUBECONFIG=$KUBECONFIG
                          kubectl set image deployment/todoappproject-deployment todoappproject-container=${DOCKER_HUB_REPO}/${IMAGE_NAME}:latest --record
                          kubectl rollout restart deployment/todoappproject-deployment
                        '''
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'The pipeline failed.'
        }
        success {
            echo 'Deployment succeeded!'
        }
    }
}
