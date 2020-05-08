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
                branch 'master'
            }
            steps{
                script {
                    app = docker.build ("mfaisal/train-schedule")
                    app.inside{
                        sh 'echo $(curl localhost:8080)'
                    }
                }

            }


        }
        stage('pushing to docker hub'){
            when {
                branch 'master'
            }
            steps{
                script{
                docker.withRegistry('https://registry.hub.docker.com', 'docker_hub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }

                }
                

            }
        }
        stage('deploy to production'){
            when {
                branch 'master'
            }
            steps{
                input 'deploy to production?'
                milestone(1)
                withCredentials([userNamepassword(credentialsId:'production_credentials',usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')])
                script{
                sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull mfaisal/train-schedule:${env.BUILD_NUMBER}\""
                try {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d mfaisal/train-schedule:${env.BUILD_NUMBER}\""
                }
            }
        }
    }
}
