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
			}
		}
	}

}