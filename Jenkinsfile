pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "prashantbande/simple-ml-app"
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo $(curl localhost:5000)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS'  ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull prashantbande/simple-ml-app:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS'  ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop simple-ml-app\""
                            sh "sshpass -p '$USERPASS'  ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm simple-ml-app\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS'  ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name simple-ml-app -p 5000:8080 -d prashantbande/simple-ml-app:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
