properties([
  buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '14', numToKeepStr: '')),
  parameters([
    string(name: 'EMAIL_NOTIFICATION', defaultValue: '', description: 'This is a comma separated list of email addresses to be notified when the build fails'),
    string(name: 'APP_DISPLAY_NAME', defaultValue: '', description: 'This will be displayed on the mobile device after the installation of the app'), 
    booleanParam(name: 'IS_DARK_THEME', defaultValue: false, description: 'Set to true If you want Dark Theme'), 
    string(name: 'MAIN_COLOUR', defaultValue: '', description: 'hex format. Please set this value without #'), 
    string(name: 'SECONDARY_COLOUR', defaultValue: '', description: 'hex format. Please set this value without #.Ignored if Dark Theme is set to true. Optional if Dark Theme is set to false. Use only if the gym has a bright secondary colour in the brand, grey shades are highly not recommended (if your secondary colour is grey, leave this option blank)'), 
    file(name: 'APP_LOGO', description: 'Resolution: Either height or width should be greater than or equal to 600px (with aspect ratio maintained)'),
    file(name: 'APP_ICON', description: 'Dimensions: 1024px x 1024px'),
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
	sh "echo hello ${NAME_TO_PRINT}"
}
