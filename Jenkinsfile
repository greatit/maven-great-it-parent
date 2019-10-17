properties([pipelineTriggers([githubPush()])])

pipeline {
	agent any
	tools {
		maven 'maven3'
		jdk 'openjdk8'
	}
	stages {
		stage('Build, test and deploy code') {
			steps {
				sh 'mvn clean deploy'
			}
		}
		stage('Generate and deploy site') {
			steps{
				sh 'mvn site-deploy deploy:deploy'
			}
		}
	}
	post {
	    regression {
	        slackSend(channel: 'javadev', color: '#FF0000', message: "REGRESSION: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
	    }
	    fixed {
	        slackSend(channel: 'javadev', color: '#00FF00', message: "FIXED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
	    }
	}
}
