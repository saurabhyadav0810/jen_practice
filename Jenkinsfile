
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'jenkins-html-deployment'
        EC2_HOST = '65.1.64.20'
        EC2_USER = 'ubuntu'
        CONTAINER_NAME = 'html-app'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/saurabhyadav0810/jen_practice.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker tag $DOCKER_IMAGE:$BUILD_NUMBER \
                            $DOCKER_USER/$DOCKER_IMAGE:$BUILD_NUMBER

                        docker push $DOCKER_USER/$DOCKER_IMAGE:$BUILD_NUMBER

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sshagent(credentials: ['ec2-ssh-key']) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no \
                                $EC2_USER@$EC2_HOST \
                                "docker pull $DOCKER_USER/$DOCKER_IMAGE:$BUILD_NUMBER &&
                                 (docker rm -f $CONTAINER_NAME || true) &&
                                 docker run -d \
                                 --name $CONTAINER_NAME \
                                 --restart unless-stopped \
                                 -p 80:80 \
                                 $DOCKER_USER/$DOCKER_IMAGE:$BUILD_NUMBER"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins console output.'
        }
    }
}
