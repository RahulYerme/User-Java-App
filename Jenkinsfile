def ARTI_VER
def appImage
pipeline {
	environment {
        registry = "rahulyerme1234/user-app"
        registryCredential = 'dockerhub'
        }
  agent any
  tools {
    maven 'maven 3.6'
  }
  stages {
    stage('Create Version') {
      steps {
        script {
          ARTI_VER = "${env.GIT_BRANCH}-${env.BUILD_NUMBER}"
          echo "${ARTI_VER}"
        }
      }
    }
    stage('Build') {
      steps {
		script {
          sh "mvn -s settings.xml -Drevision=$ARTI_VER clean package"
        }
      }
    }
    stage('Publish test and Code Coverage') {
      steps {
        junit allowEmptyResults: true, skipPublishingChecks: true,
          testResults: 'target/surefire-reports/*.xml'
        publishCoverage adapters: [jacocoAdapter('target/site/jacoco/jacoco*.xml')],
          sourceFileResolver: sourceFiles('NEVER_STORE')
      }
    }
    stage('Local artifact archive') {
      steps {
        archive 'target/app*.war'
      }
    }
    stage('Deploy to nexus') {
      steps {
        script {
          sh "mvn -s settings.xml -Drevision=$ARTI_VER deploy"
        }
      }
    }
    stage('build docker image') {
      steps {
        script {
		  withDockerServer([uri: 'tcp://192.168.43.134:2375']) {
            appImage = docker.build registry
		  }
        }
      }
   }
   stage('push docker image') {
      steps {
		script {
		  withDockerServer([uri: 'tcp://192.168.43.134:2375']) {
			withDockerRegistry(credentialsId: 'dockerhub') {
			  appImage.push("${env.BUILD_NUMBER}")
			  appImage.push("latest")
			}
		  }
        }
      }
    }
  }
  post {
    success {
      office365ConnectorSend color: '#00cc00', message: "Success  ${JOB_NAME} build_number:${BUILD_NUMBER}, branch:${env.GIT_BRANCH} url:(<${BUILD_URL}>)", status: 'SUCCESS', webhookUrl: "${env.TEAMS_WEBHOOK}"
    }
    failure {
      office365ConnectorSend color: '#fc2c03', message: "Failed  ${JOB_NAME} build_number:${BUILD_NUMBER}, branch:${env.GIT_BRANCH} url:(<${BUILD_URL}>)", status: 'FAILED', webhookUrl:"${env.TEAMS_WEBHOOK}"
    }
  }
}
