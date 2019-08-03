node {
    def server = Artifactory.server('trainjenkins.jfrog.io')
    def buildInfo = Artifactory.newBuildInfo()
    def rtMaven = Artifactory.newMavenBuild()
    
    
    stage ('Checkout & Build') {
        git url: 'https://github.com/creativetech123/jfrog-maven.git'
    }
 
    stage ('Unit Test') {
        rtMaven.tool = 'maven' // Tool name from Jenkins configuration
        rtMaven.run pom: 'pom.xml', goals: 'clean compile test'
    }
    
    withSonarQubeEnv(credentialsId: 'sonarqubeid') {
        rtMaven.tool = 'maven' // Tool name from Jenkins configuration
        rtMaven.run pom: 'pom.xml', goals: 'sonar:sonar'
    
    }
    stage("Quality Gate"){
        def ceTask
        timeout(time: 1, unit: 'MINUTES') {
          waitUntil {
            ceTask = jsonParse(url, sonarBasicAuth)
            echo ceTask.toString()
            return "SUCCESS".equals(ceTask["task"]["status"])
          }
        }
        url = new URL(sonarServerUrl + "/api/qualitygates/project_status?analysisId=" + ceTask["task"]["analysisId"])
        def qualitygate =  jsonParse(url, sonarBasicAuth)
        echo qualitygate.toString()
        if ("ERROR".equals(qualitygate["projectStatus"]["status"])) {
          error  "Quality Gate Failure"
        }
        echo  "Quality Gate Success"
      }
    
    
    stage ('Artifactory configuration') {
        // Obtain an Artifactory server instance, defined in Jenkins --> Manage..:
         
        rtMaven.tool = 'maven' // Tool name from Jenkins configuration
        rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server
        rtMaven.deployer.deployArtifacts = false // Disable artifacts deployment during Maven run
     }
    
    
            
    stage ('Install') {
        rtMaven.run pom: 'pom.xml', goals: 'install', buildInfo: buildInfo
     }
 
    stage ('Deploy') {
        rtMaven.deployer.deployArtifacts buildInfo
    }
        
    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }
       
}
