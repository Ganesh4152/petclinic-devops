pipeline {

    agent any

    environment {

        IMAGE_NAME = "Ganesh4152/petclinic-devops"

        TAG = "${BUILD_NUMBER}"

        CONTAINER = "petclinic"

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
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$TAG .'
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
                    echo "$DOCKER_PASS" | docker login \
                    -u "$DOCKER_USER" \
                    --password-stdin

                    docker push $IMAGE_NAME:$TAG

                    docker tag $IMAGE_NAME:$TAG $IMAGE_NAME:latest

                    docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {

                sshagent(['ec2-ssh-key']) {

                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@13.126.39.120 << EOF

                    docker pull $IMAGE_NAME:latest

                    docker stop $CONTAINER || true

                    docker rm $CONTAINER || true

                    docker run -d \
                    --name $CONTAINER \
                    -p 8080:8080 \
                    $IMAGE_NAME:latest

                    exit
EOF
                    '''
                }

            }
        }

    }

}
