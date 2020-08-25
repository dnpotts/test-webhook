def github_credentials = "GITHUB_TOKEN"

pipeline {

	agent { 
		node { 
			label 'master'
		}
	}
	
	environment {
		CURRENTBUILD_DISPLAYNAME= "test-webhoook build #$BUILD_NUMBER"
		CURRENT_BUILDDESCRIPTION = "test-webhook build #$BUILD_NUMBER"
		GITHUB_TOKEN = credentials('GITHUB_TOKEN')
	
	}
	
	options {
		timestamps()
		disableConcurrentBuilds()
		buildDiscarder(logRotator(numToKeepStr: '1000'))
	}
	
	triggers {
		githubPush()
		githubPullRequests (
			preStatus: true,
			triggerMode: "HEAVY_HOOKS",
			events: [
				open, commitChanged
			]
		)
	}
	
	stages {
	
		stage('Checkout Code'){
		
			steps {
				echo "env:"
				echo bat(returnStdout: true, script: 'set')
			
				script {
					currentBuild.displayName = "${env.CURRENTBUILD_DISPLAYNAME}"
					currentBuild.description = "${env.CURRENT_BUILDDESCRIPTION}"
					
					checkout([$class: 'GitSCM', branches: [[name: "${env.GITHUB_PR_HEAD_SHA}"]], doGenerateSubmoduleConfigurations: false, extensions: [], gitTool: 'GIT', submoduleCfg: [], userRemoteConfigs: [[credentialsId: "#{GITHUB_TOKEN}", url: 'https://github.com/dnpotts/test-webhook.git']]])
				}
			}
		}
		
		stage('Puppet Linting / Syntax Validation'){
		
			steps {
				powershell 'pdk validate --format=junit:report.xml --format=text:log.txt'
				powershell 'ls'
				archiveArtifacts artifacts: 'report.xml', fingerprint: true
			}
		}
	
	}
	
	post {
		
		always {
			script {
				
				echo "updating github build status"
				step([
					$class: 'GitHubCommitStatusSetter',
					reposSource: [$class: "ManuallyEnteredRepositorySource", url: "${GIT_URL}"],
					commitShaSource: [$class: "ManuallyEnteredShaSource", sha: "${GITHUB_PR_HEAD_SHA}"],
					errorHandlers: [[$class: 'ShallowAnyErrorHandler']],
					statusResultSource: [
					  $class: 'ConditionalStatusResultSource',
					  results: [
						[$class: 'BetterThanOrEqualBuildResult', result: 'SUCCESS', state: 'SUCCESS', message: "${currentBuild.description} passed validation"],
						[$class: 'BetterThanOrEqualBuildResult', result: 'FAILURE', state: 'FAILURE', message: "branch ${GITHUB_PR_SOURCE_BRANCH} failed validation, cannot merge to ${GITHUB_PR_TARGET_BRANCH}"],
						[$class: 'AnyBuildResult', state: 'FAILURE', message: 'Loophole']
					  ]
					]
				  ])
					  
			}
			  
		}
		
	}
	
}