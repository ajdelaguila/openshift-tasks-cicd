node('maven') {
  stage('Building project') {
    println "Building project"
  }
  stage('Deploying to TEST') {
    println "Deploying new version to tasks"
  }
   stage('Promoting to LIVE') {
    input "Promote new version to LIVE?"
    println "Deploying new version in Live"
  }
}
