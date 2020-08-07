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
		buildDiscarder(logRotator(numToKeepStr: '3'))
	}
	
	triggers {
		GenericTrigger (
			genericVariables: [
				[key: 'action', value: '$.action'],
				[key: 'user', value: '$.pull_request.user.login'],
				[key: 'pr_url', value: '$.pull_request.url'],
				[key: 'pr_src_sha', value: '$.pull_request.head.sha'],
				[key: 'pr_src_ref', value: '$.pull_request.head.ref'],
			],
			token: 'abc123',
			printContributedVariables: true,
			printPostContent: true,
			silentResponse: false
		)
	}
	
	stages {
		stage('Checkout Code'){
			steps {
				echo "state: ${state}"
				echo "user: ${user}"
				echo "pr_url: ${pr_url}"
				echo "pr_src_sha: ${pr_src_sha}"
				echo "pr_src_ref: ${pr_src_ref}"
				
				echo "env:"
				echo bat(returnStdout: true, script: 'set')
				
				script {
					currentBuild.displayName = "${env.CURRENTBUILD_DISPLAYNAME}"
					currentBuild.description = "${env.CURRENT_BUILDDESCRIPTION}"
					
					checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], gitTool: 'GIT', submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'GITHUB_TOKEN', url: 'https://github.com/dnpotts/test-webhook.git']]])
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
	
}