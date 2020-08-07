def github_credentials = "GITHUB_TOKEN"
def skipBuild = true

def updateGithubCommitStatus(build, repoUrl, commitSha, skipBuild) {
	if (skipBuild == false){
	  step([
		$class: 'GitHubCommitStatusSetter',
		reposSource: [$class: "ManuallyEnteredRepositorySource", url: repoUrl],
		commitShaSource: [$class: "ManuallyEnteredShaSource", sha: commitSha],
		errorHandlers: [[$class: 'ShallowAnyErrorHandler']],
		statusResultSource: [
		  $class: 'ConditionalStatusResultSource',
		  results: [
			[$class: 'BetterThanOrEqualBuildResult', result: 'SUCCESS', state: 'SUCCESS', message: build.description],
			[$class: 'BetterThanOrEqualBuildResult', result: 'FAILURE', state: 'FAILURE', message: build.description],
			[$class: 'AnyBuildResult', state: 'FAILURE', message: 'Loophole']
		  ]
		]
	  ])
  }
}

pipeline {

	agent { 
		node { 
			label 'master'
		}
	}
	
	parameters {
		string(name: 'action', defaultValue: '', description: '')
		string(name: 'user', defaultValue: '', description: '')
		string(name: 'pr_url', defaultValue: '', description: 'pull request URL')
		string(name: 'pr_src_sha', defaultValue: '', description: 'the commitId (sha) of the pull request commit to validate')
		string(name: 'pr_src_ref', defaultValue: '', description: 'the source branch of the pull request')
		string(name: 'repo_owner', defaultValue: '', description: '')
		string(name: 'repo_name', defaultValue: '', description: '')
		string(name: 'repo_url', defaultValue: '', description: '')
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
				[key: 'repo_owner', value: '$.pull_request.head.repo.owner.login'],
				[key: 'repo_name', value: '$.pull_request.head.repo.name'],
				[key: 'repo_url', value: '$.pull_request.head.repo.url']
			],
			token: 'abc123',
			printContributedVariables: true,
			printPostContent: true,
			silentResponse: false
		)
	}
	
	stages {
		stage('parse action'){
			skipBuild = false
		}
	
		stage('Checkout Code'){
			when {
				expression {
					!skipBuild
				}
			}
		
			steps {
				echo "action: ${action}"
				echo "user: ${user}"
				echo "pr_url: ${pr_url}"
				echo "pr_src_sha: ${pr_src_sha}"
				echo "pr_src_ref: ${pr_src_ref}"
				echo "repo_name: ${repo_name}"
				echo "repo_name: ${repo_url}"
				echo "repo_name: ${repo_owner}"
				
				echo "env:"
				echo bat(returnStdout: true, script: 'set')
				
				script {
					currentBuild.displayName = "${env.CURRENTBUILD_DISPLAYNAME}"
					currentBuild.description = "${env.CURRENT_BUILDDESCRIPTION}"
					
					checkout([$class: 'GitSCM', branches: [[name: "${pr_src_sha}"]], doGenerateSubmoduleConfigurations: false, extensions: [], gitTool: 'GIT', submoduleCfg: [], userRemoteConfigs: [[credentialsId: "#{GITHUB_TOKEN}", url: 'https://github.com/dnpotts/test-webhook.git']]])
				}
			}
		}
		
		stage('Puppet Linting / Syntax Validation'){
			when {
				expression {
					!skipBuild
				}
			}
			
			steps {
				powershell 'pdk validate --format=junit:report.xml --format=text:log.txt'
				powershell 'ls'
				archiveArtifacts artifacts: 'report.xml', fingerprint: true
			}
		}
	
	}
	
	post {
		//success {
			
		
			//githubNotify status: "SUCCESS", description: "Puppet //pull request syntax validation was successful", //credentialsId: "${GITHUB_TOKEN}", account: //"${repo_owner}", repo: "${repo_name}
		//}
		//unsuccessful {
		
		//}
		always {
			updateGithubCommitStatus build: currentBuild, repoUrl: "${repoUrl}", commitSha: "${pr_src_sha}", skipBuild: ${skipBuild}
		}
		
	}
	
}