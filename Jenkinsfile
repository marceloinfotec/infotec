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
        stage('Deploy To QA') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$kube_ip \"kubectl apply -f 'https://github.com/marceloinfotec/infotec/blob/master/despliegue.yml'\""
                        //sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull gilardoni72/despliegue:${env.BUILD_NUMBER}\""
                        try {
                          //  sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop despliegue\""
                          //  sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm despliegue\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                      //  sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name despliegue -p 8081:80 -d gilardoni72/despliegue:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
       stage('Deploy To kubernetes') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Kubernetes?'
                milestone(2)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$kube_ip \"kubectl apply -f 'despliegue.yml'\""
                        try {
                          //  sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop despliegue\""
                          //  sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm despliegue\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                       // sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name despliegue -p 8081:80 -d gilardoni72/despliegue:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}    
