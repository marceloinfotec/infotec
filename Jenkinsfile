pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("gilardoni72/despliegue")
                    app.inside {
                        sh 'echo $(curl localhost:80)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
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
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "docker pull gilardoni72/despliegue:${env.BUILD_NUMBER}\"
                        try {
                            sh "docker stop despliegue\"
                            sh "docker rm despliegue\"
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "docker run --restart always --name despliegue -p 80:80 -d gilardoni72/despliegue:${env.BUILD_NUMBER}\"
                    }
                }
            }
        }
    }  
}    
