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
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("ryancmcrobie/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:3000)'
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
                script {
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@$prod_ip \"docker pull ryancmcrobie/train-schedule:${env.BUILD_NUMBER}\""
                    try {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@$prod_ip \"docker stop train-schedule\""
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@$prod_ip \"docker rm train-schedule\""
                    } catch (err) {
                        echo: 'caught error: $err'
                    }
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@$prod_ip \"docker run --restart always --name train-schedule -p 3000:3000 -d ryancmcrobie/train-schedule:${env.BUILD_NUMBER}\""
                }
            }
        }
    }
}

        
      

