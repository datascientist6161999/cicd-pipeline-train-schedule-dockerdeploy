pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image'){
            when {
                branch 'solution-abhimireddy'
            }
            steps{
                echo 'Building Image...'
                script { // allows to put in script syntax in the block
                    // builds docker image with the username and image tag
                    app = docker.build("abhimireddydocker/train-schedule")
                    app.inside {
                        sh 'echo $(curl locahost:8080)' // spoke test to check for image quality
                    }
                }
            }
        }
        stage('Push Docker Image'){
            when {
                branch 'solution-abhimireddy'
            }
            steps{
                echo 'Pushing Image...'
                script{
                    // setting up docker registry url and login details
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("env.BUILD_NUMBER") // pushing build number to the docker hub account above
                        app.push("latest") // pushing latest image to the docker hub account above
                    }
                }
            }
        }
        stage('DeployToProduction'){
            when {
                branch 'solution-abhimireddy'
            }
            steps{
                echo 'Deploying To Production'
                input 'Do you want to deploy to production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId:'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]){
                    script{
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull abhimireddydocker/train-schedule:latest\""
                        try{
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        }
                        catch(err){
                            echo: 'Caught Error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d abhimireddydocker/train-schedule\""
                    }
                }
            }
        }
    }
}