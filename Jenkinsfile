pipeline {

    agent any

    environment {
        IMAGE_NAME = "shrisha21/todo-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Application') {
            steps {
                echo "Checking out application code..."

                git credentialsId: 'github-creds',
                    url: 'https://github.com/Shrishaks/todo-app-devops.git',
                    branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        echo "Pushing Docker image..."
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout
                    '''
                }
            }
        }

        stage('Checkout Kubernetes Manifests') {
            steps {
                dir('k8s-manifests') {
                    echo "Checking out Kubernetes manifests..."

                    git credentialsId: 'github-creds',
                        url: 'https://github.com/Shrishaks/todo-app-devops.git',
                        branch: 'main'
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                dir('k8s-manifests') {
                    sh '''
                        echo "======================================"
                        echo "Before image update:"
                        echo "======================================"

                        cat deploy/deploy.yaml

                        echo ""
                        echo "Updating Docker image..."

                        sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" deploy/deploy.yaml

                        echo ""
                        echo "======================================"
                        echo "After image update:"
                        echo "======================================"

                        cat deploy/deploy.yaml
                    '''
                }
            }
        }

        stage('Commit and Push Manifest') {
            steps {
                dir('k8s-manifests') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'github-creds',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )
                    ]) {
                        sh '''
                            git config user.name "Jenkins"
                            git config user.email "jenkins@localhost"

                            git add deploy/deploy.yaml

                            if git diff --cached --quiet; then
                                echo "No changes to commit."
                            else
                                git commit -m "Update image to ${IMAGE_TAG}"

                                git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Shrishaks/todo-app-devops.git HEAD:main
                            fi
                        '''
                    }
                }
            }
        }
    }

    post {

        success {
            echo "======================================"
            echo "CI/CD PIPELINE COMPLETED SUCCESSFULLY"
            echo "======================================"
            echo "Docker Image: ${IMAGE_NAME}:${IMAGE_TAG}"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo "CI/CD PIPELINE FAILED"
            echo "======================================"
            echo "Check the failed stage in Console Output."
            echo "======================================"
        }
    }
}
