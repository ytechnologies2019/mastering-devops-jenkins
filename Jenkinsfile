
pipeline {
    agent any

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
                sh 'docker login -u yinmonphyo -p ymmp@1234'
                sh 'docker container commit node yinmonphyo/node'
                sh 'docker push yinmonphyo/node '
            }
        }
     stage ('Deployment') {
        steps {
            script {
            def remote = [:]
            remote.name = 'prod-srv'
            remote.host = '13.82.4.25'
            remote.user = 'student'
            remote.password = 'Ytechnologies@123!@#'
            remote.allowAnyHosts = true
            stage('pull') {
                sshCommand remote: remote, command: "sudo docker login -u yinmonphyo -p ymmp@1234"
                sshCommand remote: remote, command: "sudo docker pull yinmonphyo/node"
                }
            stage('prod-build') { 
                sshCommand remote: remote, command: "sudo docker stop node && docker rm node"
                sshCommand remote: remote, command: "sudo docker run -dp 3000:3000 --name node yinmonphyo/node "
                }
            }
        }
    }

    }
    post { 
        always { 
            sh 'docker stop node'
            sh 'docker system prune -fa'
        }
    }
}