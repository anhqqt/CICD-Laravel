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
		stage('Get Laravel') {
			steps {
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
						sshCommand remote: remote, command: "export PROJECT_NGINX=$PROJECT_NGINX:$BUILD_ID"
						sshCommand remote: remote, command: "export PROJECT_PHP=$PROJECT_PHP:$BUILD_ID"
						// Delete old containers
						sshCommand remote: remote, command: "docker-compose down"
						// Delete old images
						sshCommand remote: remote, command: "echo y | docker image prune -a"
						// Up new containers
						sshCommand remote: remote, command: "docker-compose up -d"
						// Migrate database in Laravel
						sshCommand remote: remote, command: "docker exec php bash -c \"cd /home/cicd-laravel && php artisan migrate\""
					}
				}
			}
		}
	}

}