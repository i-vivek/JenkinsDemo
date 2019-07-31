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
 }

def printHelloWorld() {
	sh "echo hello world"
}

def printHelloNode() {
	sh "echo hello node"
}
