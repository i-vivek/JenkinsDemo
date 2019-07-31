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
	stages {
    stage('print') {
      steps {
        printHelloNode()
      }
    }
  }
}

def printHelloWorld() {
	sh "echo hello world"
}

def printHelloNode() {
	sh "echo hello node"
}
