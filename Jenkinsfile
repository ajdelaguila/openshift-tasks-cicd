node('maven') {
  // Start session with the service account jenkins which is the one configured by default for this builder
  openshift.withCluster() {
    stage('Building project') {
      // All commands in this closure will be perform in the project 'tasks-test'
      openshift.withProject( 'tasks-test' ) {
        // Get the build config for the application
        def bc = openshift.selector('bc', 'tasks')
        // Trigger the build, if it success, the new image will be pushed to the imagestream and tagged as 'latest'
        def buildInfo = bc.startBuild()
        // Follow the output of the build process until it finishes
        buildInfo.logs('-f')
      }
    }
    stage('Deploying to TEST') {
      // All commands in this closure will be perform in the project 'tasks-test'
      openshift.withProject( 'tasks-test' ) {
        // Let's make the image ready to deploy as the new application in TEST
        openshift.tag('tasks:latest', 'tasks:test')
        // Get the deployment config for the application
        def dc = openshift.selector('dc', 'tasks')
        // Since the deployment config is already pointing to a the tag 'test', deploy the new version without changing anything
        dc.rollout().latest()
        // Get the information of the replica controller created for the last deployment
        def latestDeploymentVersion = dc.object().status.latestVersion
        // Get the replica controller object
        def rc = openshift.selector('rc', "tasks-${latestDeploymentVersion}")
        // Loop until its content returns 'true' at least once
        rc.untilEach(1){
            // Refresh the information of the replica controller
            def rcMap = it.object()
            // If the number of replicas already started matches the desired number, return 'true'
            return (rcMap.status.replicas.equals(rcMap.status.readyReplicas))
        }
      }
    }
     stage('Promoting to LIVE') {
      // Wait until the test deployment has been validated before continueing
      input "Promote new version to LIVE?"
      // All commands in this closure will be perform in the project 'tasks-live'
      openshift.withProject( 'tasks-live' ) {
        // There is no declarative method to import the new image into the live project. We are referring the image locally so if the test project is eventually deleted all images will remain in the live project.
        openshift.raw( 'import-image', 'tasks:live', "--from='docker-registry.default.svc:5000/tasks-test/tasks:latest'", '--confirm', "--reference-policy='local'" )
        // Get the deployment config for the application
        def dc = openshift.selector('dc', 'tasks')
        // Since the deployment config is already pointing to a the tag 'live', deploy the new version without changing anything
        dc.rollout().latest()
        // Get the information of the replica controller created for the last deployment
        def latestDeploymentVersion = dc.object().status.latestVersion
        // Get the replica controller object
        def rc = openshift.selector('rc', "tasks-${latestDeploymentVersion}")
        // Loop until its content returns 'true' at least once
        rc.untilEach(1){
            // Refresh the information of the replica controller
            def rcMap = it.object()
            // If the number of replicas already started matches the desired number, return 'true'
            return (rcMap.status.replicas.equals(rcMap.status.readyReplicas))
        }
      }
    }
  }
}
