pipeline {
	agent any
	
	environment {
		PROJECT_GITHUB = 'https://github.com/sheid1309/CICD-Laravel.git'
		PROJECT_BRANCH = 'master'
		PROJECT_NGINX = 'sheid1309/laravel-nginx'
		PROJECT_PHP = 'sheid1309/laravel-php'
		PROJECT_SERVER = '52.163.60.66'
		MS_TEAMS_WEBHOOK = 'https://outlook.office.com/webhook/81a32ad2-372b-4e55-8dc7-484600fbd0ef@c14b46fc-2780-4bee-bcfa-e3f5a1c337b9/JenkinsCI/a37adecd236e44b0a3c25aa9f07537ae/1d2f37c9-4291-4b2e-8a54-e85aa782af39'
	}
	
	stages {
		stage('Get Code') {
			steps {
				// Notify to Slack
				slackSend message: """Get Code Started \n Job name: ${env.JOB_NAME} \n Build: ${env.BUILD_NUMBER} \n Check build output at (<${env.BUILD_URL}|${env.BUILD_URL}>)"""
				
				// Notify to MS Teams
				office365ConnectorSend message: "Last status of Build #${env.BUILD_NUMBER}", status: "Get Code Started", webhookUrl:"${MS_TEAMS_WEBHOOK}",
				factDefinitions: [[name: "Start time", template: "${BUILD_TIMESTAMP}"],
                                  [name: "Author", template: "N/A"]]
				
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
				slackSend message: """Build Image Started \n Job name: ${env.JOB_NAME} \n Build: ${env.BUILD_NUMBER} \n Check build output at (<${env.BUILD_URL}|${env.BUILD_URL}>)"""
				
				// Notify to MS Teams
				office365ConnectorSend message: "Last status of Build #${env.BUILD_NUMBER}", status: "Build Image Started", webhookUrl:"${MS_TEAMS_WEBHOOK}",
				factDefinitions: [[name: "Start time", template: "${BUILD_TIMESTAMP}"],
                                  [name: "Author", template: "N/A"]]
				
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
				slackSend message: """Build Deploy Started \n Job name: ${env.JOB_NAME} \n Build: ${env.BUILD_NUMBER} \n Check build output at (<${env.BUILD_URL}|${env.BUILD_URL}>)"""
				
				// Notify to MS Teams
				office365ConnectorSend message: "Last status of Build #${env.BUILD_NUMBER}", status: "Build Deploy Started", webhookUrl:"${MS_TEAMS_WEBHOOK}",
				factDefinitions: [[name: "Start time", template: "${BUILD_TIMESTAMP}"],
                                  [name: "Author", template: "N/A"]]
				
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
				
				// Notify to Slack
				slackSend color: "good", message: """Build Deploy Finished \n Job name: ${env.JOB_NAME} \n Build: ${env.BUILD_NUMBER} \n Check Front-end URL at (<http://${env.PROJECT_SERVER}|http://${env.PROJECT_SERVER}/>)"""	
				
				// Notify to MS Teams
				office365ConnectorSend message: "Last status of Build #${env.BUILD_NUMBER}", status: "Build Deploy Finished", color: "#1FFF00", webhookUrl:"${MS_TEAMS_WEBHOOK}",
				factDefinitions: [[name: "Start time", template: "${BUILD_TIMESTAMP}"],
                                  [name: "Author", template: "N/A"]]
			}
		}
	}

}