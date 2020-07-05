pipeline {
    agent any
    stages {
        stage('Construir') {
            steps {
                echo 'Running build automation'
                }
        }
        stage('Contruir Imagen Docker') {
            when {
                branch 'desarrollo'
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
        stage('Push Imgagen Docker') {
            when {
                branch 'desarrollo'
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
        stage('Deploy a Desarollo') {
            when {
                branch 'desarrollo'
          
            }
            steps {
                input 'Deploy a Desarrollo ip 172.18.121.128?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'dev_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                          sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip \"docker pull gilardoni72/despliegue:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip \"docker stop despliegue\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip \"docker rm despliegue\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                           sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip \"docker run --restart always --name despliegue -p 8081:80 -d gilardoni72/despliegue:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
       stage('Deploy A Produccion NO PERMITIDO') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy A Produccion?'
                milestone(2)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        //sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$kube_ip \"kubectl run despliegue --image=gilardoni72/despliegue:${env.BUILD_NUMBER} --port=80 --labels='app=despliegue' \""
                        //sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$kube_ip \"kubectl apply -f 'despliegue.yml'\""
                        try {
                          //  sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop despliegue\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$kube_ip \"kubectl delete pod despliegue\""
                        } catch (err) {
                            echo: 'caught error: $err'
 
                        }
                        
                         sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$kube_ip \"kubectl run despliegue --image=gilardoni72/despliegue:${env.BUILD_NUMBER} --port=80 --labels='app=despliegue' \""
                          
                    } 
                }
            }
        }
     }
}    
