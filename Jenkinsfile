pipeline {
	parameters {
		string(name: "MVN_VERSION", defaultValue: "3.8.5", description: "Maven version")
		string(name: "JAVA_VERSION", defaultValue: "openjdk-11", description: "Java version")
	}
	environment {
		JENKINSFILE_PLUGIN_GITHUB_PERSONAL_ACCESS_TOKEN = credentials('great-bot-github-token')
		GNUPGHOME = '/gpg-keyring/.gnupg'
		GPG_PASSPHRASE = credentials('gpg.passphrase')
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
      mountPath: /maven-repository
      subPath: .m2/repository
    - name: gnupg
      mountPath: /gpg-keyring
  imagePullSecrets:
  - name: regcred
  volumes:
  - name: m2-home
    persistentVolumeClaim:
      claimName: worker-cache
  - name: gnupg
    persistentVolumeClaim:
      claimName: worker-gpg
"""
		}
	}
	stages {
		stage('Build, test and deploy code') {
			steps {
				configFileProvider([configFile(fileId: '650e4889-d136-4524-a21c-6314f246a9f5', variable: 'MAVEN_SETTINGS_XML')]) {
					container('maven') {
						sh 'gpgconf --launch gpg-agent'
						sh 'mvn -s $MAVEN_SETTINGS_XML -Pci -Dgpg.homedir=$GNUPGHOME -Dgpg.passphrase=$GPG_PASSPHRASE clean deploy'
					}
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
