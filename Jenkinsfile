
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'saurabhyadav0810/jen_practice'
        DEPLOY_SERVER = '172.31.9.156'
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME:latest .
                '''
            }
        }

      stage('Deploy to EC2') {
    steps {
        sshagent(credentials: ['ec2-ssh-key']) {
            sh '''
                ssh -o StrictHostKeyChecking=accept-new \
                    ubuntu@$DEPLOY_SERVER "
                        sudo docker pull saurabhyadav0810/jen_practice:latest &&
                        (sudo docker rm -f jen_practice || true) &&
                        sudo docker run -d \
                            --name jen_practice \
                            --restart unless-stopped \
                            -p 80:80 \
                            saurabhyadav0810/jen_practice:latest
                    "
            '''
        }
    }
}

       