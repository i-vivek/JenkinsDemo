properties([
  buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '14', numToKeepStr: '')),
  parameters([
    string(name: 'NAME_TO_PRINT', defaultValue: '', description: 'Demo name'),
  ])
])
pipeline {
  agent any
  stages {
    stage('print') {
      steps {
        printHelloWorld()
      }
    }
  }
}

node {
	stage('print') {
		printHelloNode()
    }
    stage('print custome name') {
    	printCustomName()
    }
 }

def printHelloWorld() {
	sh "echo hello world"
}

def printHelloNode() {
	sh "echo hello node"
}

def printCustomName() {
	sh "echo hello ${ORGANISATION_ID}"
}
