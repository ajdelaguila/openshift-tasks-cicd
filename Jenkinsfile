node('maven') {
  openshift.withCluster() {
    stage('Building project') {
      // Trigger the build, if it success, the new image will be pushed to the imagestream and tagged as 'latest'
      //openshift.withProject( 'tasks-test' ) {
      //  def bc = openshift.selector('bc', 'tasks')
      //  def buildInfo = bc.startBuild()
      //  buildInfo.logs('-f')
      //}
    }
    stage('Deploying to TEST') {
      openshift.withProject( 'tasks-test' ) {
        // Let's make the image ready to deploy as the new application in TEST
        openshift.tag('tasks:latest', 'tasks:test')
        //def dc = openshift.selector('dc', 'tasks')
        def newDc = openshift.rollout('latest', 'dc/tasks')
        def latestDeploymentVersion = newDc.object().status.latestVersion
        def rc = openshift.selector('rc', "tasks-${latestDeploymentVersion}")
        rc.untilEach(1){
            def rcMap = it.object()
            return (rcMap.status.replicas.equals(rcMap.status.readyReplicas))
        }
      }
    }
     stage('Promoting to LIVE') {
      input "Promote new version to LIVE?"
      println "Deploying new version in Live"
    }
  }
}
