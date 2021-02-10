

node ('any') {

    def forcePublish             = false
    def autoDeployBranch         = false
    def isPublishBranch          = false
    def versionSuffix            = ""
    def assemblyVersion          = ""
    def nugetPackageArray        = [] as String[]
    def testAssemblies           = [] as String[]
    def projectsToPublish        = [] as String[]
    def deploymentImageName      = "support-web"
    def deploymentNamespace      = "support"
    def hotfixRegex              = "hotfix\\/.*"
    def qaRegex                  = "qa\\d?"

    def dockerPublishSettings = [
      [imageName: "Calc", dockerFile: "Dockerfile"],
      
    ]

    def dockerDeploymentSettings = [
      [deploymentName: "Calc", deploymentContext: "${env.BRANCH_NAME}", deploymentNamespace: "support", containerName: "calc", deploymentImageName: "calc"],
      
    ]

    switch(env.BRANCH_NAME) {

        case 'develop':
            versionSuffix       = ""
            isPublishBranch     = true
            autoDeployBranch    = true
        break

        case ~/$qaRegex/:
            versionSuffix       = ""
            isPublishBranch     = true
            autoDeployBranch    = true
        break

        case 'master':
            versionSuffix       = ""
            isPublishBranch     = true
            autoDeployBranch    = false
        break

        case 'prod':
            versionSuffix       = ""
            isPublishBranch     = true
            autoDeployBranch    = false
        break

        default:
            versionSuffix       = ""
            isPublishBranch     = false
            autoDeployBranch    = false
        break
    }

    def validBuildBranches = '.*(master|qa\\d*|release|develop|prod|PR-\\d*).*'
    def matched = (env.BRANCH_NAME ==~ validBuildBranches)

    if(!matched) {
      echo "Skip build for branch: '${env.BRANCH_NAME}'. Aborting Jenkins build with success."
      echo "Builds will run for: master, qa, release, develop branches, and pull requests."
      return
    }

    stage ('Checkout') {
      checkout scm
      shortCommit = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
    }

    stage ('Install & Build') {
      sh 'cat /home/jenkins/npmrc/.npmrc > ~/.npmrc'
	    sh 'npm cache clean -f'
      sh 'npm config set registry https://innroad.jfrog.io/innroad/api/npm/innroad-npm/'
      sh "npm install -no-cache"
      sh "npm run build"
      sh "cd server && npm install"
    }

    stage ('Run Unit Tests') {
      sh "npm test"
    }

    stage ("Publish to Docker") {
      echo "Starting Publish To Docker"
      dockerPublishSettings.each {a ->
        pushToDocker(isPublishBranch, a.imageName, shortCommit, a.dockerFile)
      }
    }

    stage('Deploy') {
      echo "Starting Deploy To Docker"
      dockerDeploymentSettings.each { a ->
        deployDockerImageV2(autoDeployBranch, a.deploymentName, a.deploymentImageName, a.deploymentContext, a.deploymentNamespace, shortCommit, a.containerName)
      }
    }
}
