def github_credentials = "GITHUB_TOKEN"
def skipBuild = true

void updateGithubCommitStatus(build, repoUrl, commitSha, skipBuild) {
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
		string(name: 'pr_base_sha', defaultValue: '', description: 'the commitId (sha) of the pull request commit to merge to')
		string(name: 'pr_base_ref', defaultValue: '', description: 'the branch of the pull request to merge to')
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
		buildDiscarder(logRotator(numToKeepStr: '100'))
	}
	
	triggers {
		GenericTrigger (
			genericVariables: [
				[key: 'action', value: '$.action'],
				[key: 'user', value: '$.pull_request.user.login'],
				[key: 'pr_url', value: '$.pull_request.url'],
				[key: 'pr_src_sha', value: '$.pull_request.head.sha'],
				[key: 'pr_src_ref', value: '$.pull_request.head.ref'],
				[key: 'pr_base_sha', value: '$.pull_request.base.sha'],
				[key: 'pr_base_ref', value: '$.pull_request.base.ref'],
				[key: 'repo_owner', value: '$.pull_request.head.repo.owner.login'],
				[key: 'repo_name', value: '$.pull_request.head.repo.name'],
				[key: 'repo_url', value: '$.pull_request.head.repo.html_url']
			],
			token: 'abc123',
			printContributedVariables: true,
			printPostContent: true,
			silentResponse: false,
			causeString: '$user submitted pull request from $pr_src_ref (sha $pr_src_sha) to $pr_base_ref (sha $pr_base_sha)'
		)
	}
	
	stages {
		stage('parse action'){
			steps {
				echo "action: ${action}"
				echo "user: ${user}"
				echo "pr_url: ${pr_url}"
				echo "pr_src_sha: ${pr_src_sha}"
				echo "pr_src_ref: ${pr_src_ref}"
				echo "pr_base_sha: ${pr_base_sha}"
				echo "pr_base_ref: ${pr_base_ref}"
				echo "repo_name: ${repo_name}"
				echo "repo_url: ${repo_url}"
				echo "repo_owner: ${repo_owner}"
				
				echo "env:"
				echo bat(returnStdout: true, script: 'set')
				
				script {
					// only run pipeline if action is "opened" or "edited"
					switch("${action}"){
						case "opened":
						case "edited":
						case "reopened":
							skipBuild = false
							break
						default:
							skipBuild = true 
							break;
					}
				}
				echo "skipBuild: ${skipBuild}"
			}
		}
	
		stage('Checkout Code'){
			when {
				expression {
					!skipBuild
				}
			}
		
			steps {
								
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
		
		always {
			script {
				if(skipBuild == false){
					echo "updating github build status"
					step([
						$class: 'GitHubCommitStatusSetter',
						reposSource: [$class: "ManuallyEnteredRepositorySource", url: "${repo_url}"],
						commitShaSource: [$class: "ManuallyEnteredShaSource", sha: "${pr_src_sha}"],
						errorHandlers: [[$class: 'ShallowAnyErrorHandler']],
						statusResultSource: [
						  $class: 'ConditionalStatusResultSource',
						  results: [
							[$class: 'BetterThanOrEqualBuildResult', result: 'SUCCESS', state: 'SUCCESS', message: "${currentBuild.description} passed validation"],
							[$class: 'BetterThanOrEqualBuildResult', result: 'FAILURE', state: 'FAILURE', message: "branch ${pr_src_ref} failed validation, cannot merge to ${pr_base_ref}"],
							[$class: 'AnyBuildResult', state: 'FAILURE', message: 'Loophole']
						  ]
						]
					  ])
					  
					 
				} else {
					echo "skipping github status update"
				}
			}
			  
		}
		
	}
	
}