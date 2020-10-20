timestamps{
    def tag="blue"
    def altTag="green"
    def routeHost="${tag}-${NAME}-${PROJECT}-prd.apps.openshift.oracle.msdigital.pro"
    node('nodejs'){
        stage('Checkout'){
           //checkout([$class: 'GitSCM', branches: [[name: '*/blue-green']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/cmotta2016/gs-spring-boot.git']]])
            checkout scm
        }//stage
        openshift.withCluster() {
            openshift.withProject("${PROJECT}-prd") {
                stage("Initialize Blue-Green Routes") {
                    if (openshift.selector("route", "${tag}-${NAME}").exists()) {
                      def activeService = openshift.raw("get route ${NAME} -o jsonpath='{ .spec.to.name }' --loglevel=4").out.trim()
                      if (activeService == "${NAME}-blue") {
                        tag = "green"
                        altTag = "blue"
                      }//if
                      routeHost = openshift.raw("get route ${tag}-${NAME} -o jsonpath='{ .spec.host }' --loglevel=4").out.trim()
                    }//if
                }//stage
                stage('Re-tagging Image'){
                    openshift.tag("${REPOSITORY}/${NAME}:latest", "${REPOSITORY}/${NAME}:${tag}")
                }//stage
                stage('Deploy PRD') {
                    echo "Applying Template PRD"
		            //openshift.apply(openshift.process(readFile(file:'template-nodejs.yml'), "--param-file=template_environments"))
                        if (openshift.selector("route", "${NAME}").exists()) {
                                    openshift.apply(openshift.process(readFile(file:'template-nodejs.yml'), "--param-file=template_environments"), "-l name!=principal")
                        } else {
                            openshift.apply(openshift.process(readFile(file:'template-nodejs.yml'), "--param-file=template_environments"))
                        }
                    echo "Starting Deployment PRD"
                    openshift.selector("dc", "${NAME}-${tag}").rollout().latest()
                    def dc = openshift.selector("dc", "${NAME}-${tag}")
                    dc.rollout().status()
                }//stage
                stage("Test Deployment") {
                    input message: "Test deployment: http://${routeHost}. Approve?", id: "approval"
                }//stage
                stage("Go Live") {
                    openshift.raw("set route-backends ${NAME} ${NAME}-${tag}=100 ${NAME}-${altTag}=0 --loglevel=4").out
                }//stage
		stage('Reduzindo HPA') {
		     openshift.raw("patch hpa ${NAME}-${altTag} --type='json' -p='[{"op": "replace", "path": "/spec/minReplicas", "value":"1"}]'").out
		}//stage
            }//withProject
        }//withCluster
    }//node
}//timestamp
