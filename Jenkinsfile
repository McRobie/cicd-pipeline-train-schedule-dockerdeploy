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
                sshPublisher (
                    failOnError: true,
                    continueOnError: false,
                    publishers: [
                        sshPublisherDesc(
                            configName: 'production',
                            transfers: [
                                sshTransfer(
                                    execCommand: "docker pull ryancmcrobie/train-schedule:${env.BUILD_NUMBER}"
				    execCommand: "docker stop train-schedule"
				    execCommand: "docker rm train-schedule"
				    execCommand: "docker run --restart always --name train-schedule -p 3000:3000 -d ryancmcrobie/train-schedule:${env.BUILD_NUMBER}"
                                )
			    ]
			)
		    ]
		)	
            }
        }
    }
}
        
      

