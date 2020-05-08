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
						}
					}
				}
			}
		}
		
		stage('Deploy') {
			steps {
				script {
					def remote = [:]
					remote.name = 'test'
					remote.host = '23.98.73.86'
					remote.user = 'matdecha'
					remote.password = 'a130993310594W'
					remote.allowAnyHosts = true
					stage('Remote SSH') {
					  sshCommand remote: remote, command: "wget https://raw.githubusercontent.com/sheid1309/CICD-Laravel/master/docker-compose.yml"
					  sshCommand remote: remote, command: "ls -la"
					}
				}
			}
		}
	}

}