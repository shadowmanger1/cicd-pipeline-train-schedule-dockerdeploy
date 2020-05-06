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
			app = docker.build("shadowmanger/train-schedule")
			app.inside {
				sh 'echo $(curl localhost:8080)'
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
			docker.withRegistry('https://registry.hub.docker.com'. 'docker_hub_login') {
				app.push("${env.BUILD_NUMBER}")
				app.push("latest")
			}
		}
	    }
        }
        stage('Deploy To Production') {
            when {
		branch 'master'
	    }
	    steps {
                withCredentials([usernamePassword(credentialsId: 'web_production', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
			script {
				sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull shadowmanger1/train-schedule:${env.BUILD_NUMBER}\""
				try {
					sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyCheking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
					sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyCheking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
				} catch(err) {
					echo: 'Error: $err'
				}
				sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d shadowmnager1/train-schedule:${env.BUILD_NUMBER}\""
                    	}
                }
            }
        }
    }
}

