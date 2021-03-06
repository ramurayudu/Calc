@Library("rayudu") _

node {

env.NODE_ENV = 'main'

    def forcePublish             = true
    def autoDeployBranch         = true
    def isPublishBranch          = true
    def versionSuffix            = ""
    def assemblyVersion          = ""
    def nugetPackageArray        = [] as String[]
    def testAssemblies           = [] as String[]
    def projectsToPublish        = [] as String[]
    def deploymentImageName      = "calc"
    def deploymentNamespace      = "loginapp"
    def hotfixRegex              = "hotfix\\/.*"
    def qaRegex                  = "qa\\d?"

    def dockerPublishSettings = [
      [imageName: "calc", dockerFile: "Dockerfile"],
      
    ]

    def dockerDeploymentSettings = [
      [deploymentName: "calc", deploymentContext: "${env.BRANCH_NAME}", deploymentNamespace: "loginapp", containerName: "calc", deploymentImageName: "calc"],
      
    ]

    switch(env.BRANCH_NAME) {

        case 'main':
            versionSuffix       = ""
            isPublishBranch     = true
            autoDeployBranch    = true
        break

        

        default:
            versionSuffix       = ""
            isPublishBranch     = false
            autoDeployBranch    = false
        break
    }

    def validBuildBranches = '.*(main|qa\\d*|release|main|prod|PR-\\d*).*'
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
      
	    sh 'npm cache clean -f'
  /*    sh 'npm config set registry https://innroad.jfrog.io/innroad/api/npm/innroad-npm/' */
      sh "npm install -no-cache"
      sh "npm run build"
     /* sh "docker login devopsinnroad.jfrog.io -u srinivas.rayudu94@gmail.com -p ${jfrog}" */
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
