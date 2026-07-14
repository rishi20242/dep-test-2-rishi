pipeline {
    agent any
    
    environment {
        IMAGE_NAME = "teslaguy007/my-app"
        TAG = "${BUILD_NUMBER}"
        IMAGE_WITH_TAG = "${IMAGE_NAME}:${TAG}"
        IMAGE_LATEST = "${IMAGE_NAME}:latest"
        DOCKER_HUB_CREDS = 'docker_credentials'
    }

    stages {
        stage('Install') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                . venv/bin/activate
                pytest
                '''
            }
        }

        stage('Build & Push') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKER_HUB_CREDS}") {
                        sh "docker build -t ${IMAGE_WITH_TAG} ."
                        sh "docker tag ${IMAGE_WITH_TAG} ${IMAGE_LATEST}"
                        sh "docker push ${IMAGE_WITH_TAG}"
                        sh "docker push ${IMAGE_LATEST}"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                withKubeConfig([credentialsId: 'minikube-config']) {
                    script {
                        if (fileExists('k8s/deployment.template.yaml')) {
                            sh "sed 's|IMAGE_PLACEHOLDER|${IMAGE_WITH_TAG}|g' k8s/deployment.template.yaml > k8s/deployment.yaml"
                            sh "cat k8s/deployment.yaml"
                            sh "kubectl apply -f k8s/deployment.yaml"
                            sh "kubectl apply -f k8s/service.yaml"
                            sh "kubectl rollout restart deployment/sample-app"
                            sh "kubectl rollout status deployment/sample-app"
                        } else {
                            error "Template file k8s/deployment.template.yaml not found!"
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            sh "rm -f k8s/deployment.yaml"
        }
    }
}
