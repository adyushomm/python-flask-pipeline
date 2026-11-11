pipeline {
    agent any

    environment {
        // Updated image name to match your Docker Hub user and repo
        IMAGE_NAME = "adyushomm/docker-demo"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Test Container') {
            steps {
                sh '''
                    docker stop test-container || true
                    docker rm test-container || true

                    # Launch temporary test container
                    docker run -d \
                        --name test-container \
                        $IMAGE_NAME:$IMAGE_TAG

                    sleep 5

                    # Test Nginx web server directly inside the test container
                    docker exec test-container \
                        python3 -c "import urllib.request; print(urllib.request.urlopen('http://127.0.0.1:80/').read().decode())"
                '''
            }

            post {
                always {
                    sh '''
                        docker stop test-container || true
                        docker rm test-container || true
                    '''
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-cred',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push $IMAGE_NAME:$IMAGE_TAG

                    docker tag \
                        $IMAGE_NAME:$IMAGE_TAG \
                        $IMAGE_NAME:latest

                    docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker stop my-app || true
                    docker rm my-app || true

                    # Deploy live container: maps host port 5000 to container port 80
                    docker run -d \
                        --name my-app \
                        -p 5000:80 \
                        --restart unless-stopped \
                        $IMAGE_NAME:$IMAGE_TAG

                    sleep 5

                    docker ps

                    # Verify app is responding on host port 5000
                    curl -f http://localhost:5000
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline Completed Successfully'
        }

        failure {
            echo 'Pipeline Failed'
        }

        always {
            sh '''
                docker stop test-container || true
                docker rm test-container || true
                docker image prune -f || true
            '''
        }
    }
}
