pipeline {
	agent any
	
	environment {
		PROJECT_GITHUB = 'https://github.com/sheid1309/CICD-Laravel.git'
		PROJECT_BRANCH = 'master'
		PROJECT_NGINX = 'sheid1309/laravel-nginx'
		PROJECT_PHP = 'sheid1309/laravel-php'
		PROJECT_SERVER = '23.98.73.86'
	}
	
	stages {
		stage('Get Code') {
			steps {
				// Notify to Slack channel
				slackSend color: "good", message: "Get Code Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
			
				// Clear current workplace cache
				sh "rm -rf *"
			
				git(url: PROJECT_GITHUB, branch: PROJECT_BRANCH)
				sh "tar -cvzf cicd-laravel.tar.gz nginx php src docker-compose.yml" 
				// No need stash as we run directly from the jenkins master
				// stash includes: 'cicd-laravel.tar.gz', name: 'cicd-laravel-project'			
			}
		}
		
		stage('Build Image') {			
			steps {
				// Notify to Slack
				slackSend message: "Build Image Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
			
				// Delete current images on Jenkins slave (currently this machine)
				// sh "echo y | docker image prune -a"

				// Copy src to image folder
				sh "cp -r src nginx/"
				script {
					dir("nginx") {
						def image = docker.build(PROJECT_NGINX)
					
						docker.withRegistry('', 'dockerhub_id') {
							image.push(BUILD_ID)
							image.push("latest")
						}
					}
				}
				
				// Copy src to image folder
				sh "cp -r src php/"
				script {
					dir("php") {
						def image = docker.build(PROJECT_PHP)
					
						docker.withRegistry('', 'dockerhub_id') {
							image.push(BUILD_ID)
							image.push("latest")
						}
					}
				}
			}
		}
		
		stage('Deploy') {
			steps {
				// Notify to Slack
				slackSend message: "Build Deploy Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
				
				script {
					withCredentials([sshUserPrivateKey(
						credentialsId: 'webserver_key',
						keyFileVariable: 'identityFile',
						passphraseVariable: '',
						usernameVariable: 'user'
					)]) {
					    def remote = [:]
						remote.name = 'server'
						remote.host = PROJECT_SERVER
						remote.user = user
						remote.identityFile = identityFile
						remote.allowAnyHosts = true
						
						// Clear old docker-compose
						sshCommand remote: remote, command: "rm -rf docker-compose.yml"
						sshCommand remote: remote, command: "wget https://raw.githubusercontent.com/sheid1309/CICD-Laravel/master/docker-compose.yml"
						// Set global var
						sshCommand remote: remote, command: "export PROJECT_NGINX=$PROJECT_NGINX:$BUILD_ID && export PROJECT_PHP=$PROJECT_PHP:$BUILD_ID && docker-compose down && echo y | docker image prune -a && docker-compose up -d"
						// Wait until container finished creating
						sshCommand remote: remote, command: "sleep 20"
						// Migrate database in Laravel
						sshCommand remote: remote, command: "docker exec php bash -c \"cd /home/cicd-laravel && php artisan migrate\""
					}
				}
				
				// Notify to Slack again
				slackSend color: "good", message: "Build Deployed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<http://23.98.73.86/|http://23.98.73.86/>))"
			}
		}
	}

}