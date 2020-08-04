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
	
	stages {
		stage('Checkout Code'){
			steps {
				powershell "write-output $env:payload"
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