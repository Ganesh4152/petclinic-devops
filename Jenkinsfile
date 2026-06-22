pipeline {

    agent any

    environment {
        IMAGE_NAME = "gani4152/petclinic-devops"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "petclinic"
        PROD_SERVER = "52.66.57.217"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-pat',
                    url: 'https://github.com/Ganesh4152/petclinic-devops.git'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    docker push $IMAGE_NAME:$IMAGE_TAG
                    docker push $IMAGE_NAME:latest

                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                sshagent(['ec2-ssh-key']) {

                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@$PROD_SERVER << EOF

                    sudo systemctl start docker

                    docker pull $IMAGE_NAME:latest

                    docker stop $CONTAINER_NAME || true

                    docker rm $CONTAINER_NAME || true

                    docker run -d \
                      --name $CONTAINER_NAME \
                      -p 8080:8080 \
                      --restart unless-stopped \
                      $IMAGE_NAME:latest

                    docker ps

                    exit
EOF
                    '''
                }
            }
        }
    }

    post {

        success {
            echo "========================================"
            echo "CI/CD Pipeline Completed Successfully"
            echo "Application Deployed Successfully"
            echo "========================================"
        }

        failure {
            echo "========================================"
            echo "CI/CD Pipeline Failed"
            echo "Check Console Output"
            echo "========================================"
        }

        always {
            cleanWs()
        }
    }
}
