try {
   timeout(time: 20, unit: 'MINUTES') {
      def appName="nodejs-mongodb-example"
      def project=""
      def tag="blue"
      def altTag="green"
      def verbose="false"

      node {
        project = env.PROJECT_NAME
        stage("Initialize") {
          sh "oc get route ${appName} -n ${project} -o jsonpath='{ .spec.to.name }' --loglevel=4 > activeservice"
          activeService = readFile('activeservice').trim()
          if (activeService == "${appName}-blue") {
            tag = "green"
            altTag = "blue"
          }
          sh "oc get route ${tag}-${appName} -n ${project} -o jsonpath='{ .spec.host }' --loglevel=4 > routehost"
          routeHost = readFile('routehost').trim()
        }

        openshift.withCluster() {
          openshift.withProject() {
            stage("Build") {
              echo "building tag ${tag}"
              def bld = openshift.startBuild("${appName}")
              bld.untilEach {
                return it.object().status.phase == "Running"
              }
              bld.logs('-f')                        
            }
  
            stage("Deploy Test") {
              openshift.tag("${appName}:latest", "${appName}:${tag}")
              def dc = openshift.selector('dc', "${appName}-${tag}")
              dc.rollout().status()
            }
  
            stage("Test") {
              input message: "Test deployment: http://${routeHost}. Approve?", id: "approval"
            }
  
            stage("Go Live") {
              sh "oc set -n ${project} route-backends ${appName} ${appName}-${tag}=100 ${appName}-${altTag}=0 --loglevel=4"
            }

          }
        }
      }
   }
} catch (err) {
   echo "in catch block"
   echo "Caught: ${err}"
   currentBuild.result = 'FAILURE'
   throw err
}          
