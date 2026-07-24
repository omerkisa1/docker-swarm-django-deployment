pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh "docker build -t omerkisa1/django-web:v1.0.${BUILD_NUMBER} -t omerkisa1/django-web:latest ."
            }
        }

        stage('Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push omerkisa1/django-web:latest"
                    sh "docker push omerkisa1/django-web:v1.0.${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Swarm') {
            steps {
                sshagent(['manager-ssh-key']) {
                    sh "scp -o StrictHostKeyChecking=no docker-stack.yml ubuntu@10.0.1.237:/home/ubuntu/docker-stack.yml"
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@10.0.1.237 'docker stack deploy -c /home/ubuntu/docker-stack.yml swarmproject'"
                }
            }
        }
    }
}
