import hudson.FilePath
import hudson.model.ParametersAction
import hudson.model.FileParameterValue
import hudson.model.Executor
properties([
  buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '14', numToKeepStr: '')),
  parameters([
    string(name: 'EMAIL_NOTIFICATION', defaultValue: 'vivek.a@clubware.co.nz', description: 'This is a comma separated list of email addresses to be notified when the build fails'),
    string(name: 'APP_DISPLAY_NAME', defaultValue: 'club01', description: 'This will be displayed on the mobile device after the installation of the app'), 
    booleanParam(name: 'IS_DARK_THEME', defaultValue: false, description: 'Set to true If you want Dark Theme'), 
    string(name: 'MAIN_COLOUR', defaultValue: '994322', description: 'hex format. Please set this value without #'), 
    string(name: 'SECONDARY_COLOUR', defaultValue: '', description: 'hex format. Please set this value without #.Ignored if Dark Theme is set to true. Optional if Dark Theme is set to false. Use only if the gym has a bright secondary colour in the brand, grey shades are highly not recommended (if your secondary colour is grey, leave this option blank)'), 
    file(name: 'APP_LOGO', description: 'Resolution: Either height or width should be greater than or equal to 600px (with aspect ratio maintained)'),
    file(name: 'APP_ICON', description: 'Dimensions: 1024px x 1024px'),
    string(name: 'CERTS_AND_PROFILES_GIT_BASE_URL', defaultValue: 'https://bitbucket.org/clubwaremobile/certsandprofiles.git', description: 'Bitbucket certs and profiles url to checkout')
  ])
])

node {
  wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm']) {
    withCredentials([
        [$class: 'UsernamePasswordMultiBinding', credentialsId: params['GIT_CREDENTIAL'], passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USER_EMAIL_ADDRESS'],
        [$class: 'StringBinding', credentialsId: params['FASTLANE_MATCH_PASSWORD'], variable: 'MATCH_PASSWORD']
      ]) {

      def errorException = null

      try {
        //A constant which needs to be accessed in multiple stages
        def bitbucketUserName="cwmobiledev"
        def branchToCheckout="develop"

        stage('Prepare Workspace') {
            cleanWs()
        }

        stage('Check Input Parameters') {
            checkInputParameters()
        }

        stage('Set build display name') {
            currentBuild.displayName = "Preview app of ${APP_DISPLAY_NAME}"
        }

        stage('Prepare environment variables') {
          // This value is required - It should be configured from Jenkins - Not from fastfile Script
          env.LC_ALL="en_US.UTF-8"
          env.LANG="en_US.UTF-8"
          // Setting this explicitly to 2.5.3 since fastlane has some issue with the ruby system (~2.6) version 
          env.RBENV_VERSION="2.5.3"

          env.CERTS_AND_PROFILES_GIT_USERNAME="${bitbucketUserName}"
          env.CERTS_AND_PROFILES_GIT_URL=CERTS_AND_PROFILES_GIT_BASE_URL.replace("://","://"+bitbucketUserName+"@")
        } 

        stage('Checkout SourceCode') {
          echo "${branchToCheckout} Branch to check out!"
          def escapedPasswordString = "${GIT_PASSWORD}".replace("@", "%40")
          cloneGitRepo("${bitbucketUserName}", "${escapedPasswordString}", "${branchToCheckout}", "${BITBUCKET_ONBOARDING_SOURCECODE_URL}")        
        }
        
        if (!(ONLY_SHOW_BRANDING_DATA.toBoolean())) {
          stage('Copy Files Into Workspace') {
            copyFilesIntoWorkspace()
          }
        }
        
        stage('Onboard Client') {
          def timeoutValueInMinutes=60
          timeout(timeoutValueInMinutes) {
              invokefastlane('onboardClient', env.WORKSPACE)
          }
        }      
      }
      catch(exception) {
        println exception
        errorException = exception
      }
      finally {
        boolean isSuccess = true
        if(errorException) {
          isSuccess = false
        }
        stage('Send Email') {
          emailext body: getEmailBody(isSuccess, env.ORGANISATION_ID),
                subject: getEmailSubject(isSuccess, env.ORGANISATION_ID),
                to: "${env.EMAIL_NOTIFICATION}"
        }
        stage('CleanUp Workspace') {
          cleanWs()
        }
        //fail the job if in case of failure
        if(errorException) {
          throw errorException
        }
      }
    }
  }
}

def checkInputParameters() {
    if (!EMAIL_NOTIFICATION) {
      error 'Empty EMAIL_NOTIFICATION! Provide a valid emailaddress'
    }
}

def cloneGitRepo(String username, String password, String branch, String url) {
    GIT_URL=url.replace("://","://"+username + ":" + password + "@")
    sh  """#!/bin/bash -l
        git clone --depth 1 --branch ${branch} "${GIT_URL}"
        """ 
}

def getEmailBody(boolean success, String orgId) {
  def colorOfTheEmailStatus="green"
  def emailStatusMessage="Success"
  if(!success) {
    colorOfTheEmailStatus="red"
    emailStatusMessage="Failed"
  }
  def emailBody = """
  <p>
  <span style="color:${colorOfTheEmailStatus}">Onboarding Client OrgId:${orgId} ${emailStatusMessage}!</span><br/><br/>
  Please check the <a href="${env.BUILD_URL}consoleFull">build log</a><br/>
  </p>
  """
  return emailBody
}

def getEmailSubject(boolean success, String orgId) {
  def emailStatusMessage="Success"
  if(!success) {
    emailStatusMessage="Failed"
  }
  return "Onboarding Client OrgId:${orgId} ${emailStatusMessage} - #${BUILD_NUMBER}!!!"
}

def invokefastlane(String lane, String workSpace) {
    sh '''#!/bin/bash -l
          cd ''' + workSpace + '''/onboardingscripts
          git show --oneline -s
          rbenv version
          bundle install
          bundle exec fastlane ''' + lane + '''
    '''
}

def copyFilesIntoWorkspace() {
  def paramsAction = currentBuild.rawBuild.getAction(ParametersAction.class);
  if (paramsAction != null) {
    for (param in paramsAction.getParameters()) {
      echo "${param.getName()}"
      if (param.getName().equals("APP_LOGO") || param.getName().equals("APP_ICON")) {
        if (! param instanceof FileParameterValue) {
            error "not a file parameter"
        }
        if (env['NODE_NAME'] == null) {
            error "no node in current context"
        }
        if (env['WORKSPACE'] == null) {
            error "no workspace in current context"
        }
        println env['NODE_NAME']
        println env['WORKSPACE']
        nodeName = env['NODE_NAME'] == 'develop' ? '(develop)' : env['NODE_NAME']
        workspace = new FilePath(Jenkins.getInstance().getComputer(nodeName).getChannel(), env['WORKSPACE']+"/onboardingscripts")
        echo "workspace = ${workspace}"
        filename = param.getOriginalFileName()
        if (filename.endsWith("png")) {
          echo "filename after copying : ${param.getName()}.png"
          file = workspace.child(param.getName()+".png")
          echo "filesize : ${param.getFile().getSize()}"
          if (param.getFile().getSize()>0) {
            file.copyFrom(param.getFile())
            echo "file : ${filename} copied to ${param.getName()}.png"
          } else {
            error "Empty ${param.getOriginalFileName()}! Provide a valid ${param.getName()}"
          }
        } else {
          error "Invalid ${param.getOriginalFileName()} format! Provide a valid ${param.getName()}. Image should only be a png file."
        }
      }
    }
    paramsAction = null
  }
}
