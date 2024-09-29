pipeline {
    agent any
    environment {     
        DOCKERHUB_LOGIN= credentials('DOCKERHUB') //EnvironmentVariableFromCredentials    
    } 

    stages {
        stage('Pull') {
            steps {
                git branch: 'main', url: 'https://github.com/ytechnologies2019/to-do-app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t getting-started-app .'
                sh 'docker run -d --name node getting-started-app'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'docker exec node npm test'
            }
        }

        stage('Create Image') {
            steps {
                sh 'echo $DOCKERHUB_LOGIN_PSW | docker login -u $DOCKERHUB_LOGIN_USR --password-stdin'
                sh 'docker container commit node yinmonphyo/node'
                sh 'docker push yinmonphyo/node'
            }
        }
        //Deployment to srv
        stage('Deployment') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'srv-login', passwordVariable: 'SERVER_PASSWORD', usernameVariable: 'SERVER_USERNAME')]) {
                        def remote = [:]
                        remote.name = 'prod-srv'
                        remote.host = '13.82.4.25'
                        remote.user = "${SERVER_USERNAME}"
                        remote.password = "${SERVER_PASSWORD}"
                        remote.allowAnyHosts = true 

                        stage('Pull') {
                            sshCommand remote: remote, command: "echo $DOCKERHUB_LOGIN_PSW | sudo docker login -u $DOCKERHUB_LOGIN_USR --password-stdin" // Consider using environment variables for this as well
                            sshCommand remote: remote, command: "sudo docker pull yinmonphyo/node"
                        }

                        stage('Prod Build') { 
                            sshCommand remote: remote, command: "sudo docker stop node || true" // Use || true to avoid failure if the container is not running
                            sshCommand remote: remote, command: "sudo docker rm node || true" // Same as above
                            sshCommand remote: remote, command: "sudo docker run -dp 3000:3000 --name node yinmonphyo/node"
                        }
                    }
                }
            }
        }
    }

    post { 
        always { 
            sh 'docker stop node || true' // Prevent errors if the container isn't running
            sh 'docker system prune -fa'
        }
    }
}
