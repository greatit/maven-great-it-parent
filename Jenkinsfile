pipeline {
	parameters {
		string(name: "MVN_VERSION", defaultValue: "3.6.3", description: "Maven version")
		string(name: "JAVA_VERSION", defaultValue: "openjdk-8", description: "Java version")
	}
	environment {
		JENKINSFILE_PLUGIN_GITHUB_PERSONAL_ACCESS_TOKEN = credentials('great-bot-github-token')
	}

	agent {
		kubernetes {
			yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: harbor.prod.internal.great-it.com/library/maven:${params.MVN_VERSION}-${params.JAVA_VERSION}
    imagePullPolicy: Always
    command:
    - cat
    env:
    - name: TZ
      value: Europe/Budapest
    tty: true
    volumeMounts:
    - name: m2-home
      mountPath: /root/.m2
  volumes:
  - name: m2-home
    hostPath:
      path: /var/lib/jenkins-prod/.m2
      type: Directory
"""
		}
	}
	stages {
		stage('Build, test and deploy code') {
			steps {
				container('maven') {
					sh 'mvn clean site deploy'
				}
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
