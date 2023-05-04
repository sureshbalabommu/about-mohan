pipeline {
	agent {
		label 'master'
	}
	tools {
		maven 'maven4'
	}
//	parameters {
//		choice (
//			choices: ['false', 'true'], 
//			description: 'Run CI Job', 
//			name: 'ci'
//			)
//		choice (
//			choices: ['false', 'true'], 
//			description: 'Run cd Job', 
//			name: 'cd'
//			)			
//	}
	environment {
		pom = readMavenPom file: 'pom.xml'
		version = "${pom.version}"
		artifactId = "${pom.artifactId}"
		env_config_id = "abou-mohan"
		ci = "true"
		cd = "true"
  	}
	stages {
		stage('load env vars') {
			steps {
				script {
					configFileProvider([configFile(fileId: "${env_config_id}", targetLocation: 'env.groovy')]) {	
					load 'env.groovy'
					}
				}
			}
		}

		stage('sonar') {
		when {
            	expression {ci == 'true'}
            }
			steps {
				withCredentials([string(credentialsId: "${sonar_id}", variable: 'sonar_token')]) {
				sh "mvn clean install  sonar:sonar -Dsonar.host.url=${sona_url}  -Dsonar.login=${sonar_token}"
				}
			}
		}
		stage('build & deploy') {
		when {
                expression {ci == 'true'}
            }
			steps {
				withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'nexus_passwd', usernameVariable: 'nexus_user')]) {
				sh "mvn clean deploy -s settings.xml"
				}
			}
		}
		stage('deploy') {
		when {
                	expression {cd == 'true'}
            	}
		agent {
			label 'ansible'
                }
			steps {
				checkout([
					$class: 'GitSCM', 
					branches: [[name: '*/master']], 
					extensions: [], 
					userRemoteConfigs: [
					[
						credentialsId: "${git_cred_id}", 
						url: "${deploy_git}"
					]
					]
				])
				sh 'ansible-playbook -i inventory.yml deploy.yml -e version=$version -e artifactId=$artifactId -e artifactory_url=$artifactory_url -e arti_path=$arti_path'
			}
		}
	}
}
