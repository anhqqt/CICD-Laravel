pipeline {
	agent any
	
	environment {
		PROJECT_GITHUB = 'https://github.com/sheid1309/CICD-Laravel.git'
		PROJECT_BRAND = 'master'
		PROJECT_NGINX = 'sheid1309/laravel-nginx'
		PROJECT_PHP = 'sheid1309/laravel-php'
		PROJECT_SERVER = '23.98.73.86'
	}
	
	stages {
		stage('Get Laravel') {
			steps {
				git(url: PROJECT_GITHUB, branch: PROJECT_BRAND)
				stash(name: 'frontend', includes: '*')
			}
		}
		
		stage('Test') {
			steps {
			
			}
		}
		
		stage('Deploy') {
			steps {
			
			}
		}
		
		stage('Test Feature') {
			steps {
			
			}
		}
	}

}