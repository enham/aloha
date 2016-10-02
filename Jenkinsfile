node {
    stage 'Build image and deploy in Dev'
    echo 'Building docker image and deploying to Dev'
    buildAloha('helloworld-msa-dev')

    stage 'Deploy to QA'
    echo 'Deploying to QA'
    deployAloha('helloworld-msa-dev', 'helloworld-msa-qa')

    stage 'Wait for approval'
    input 'Aprove to production?'

    stage 'Deploy to production'
    echo 'Deploying to production'
    deployAloha('helloworld-msa-dev', 'helloworld-msa')
}

// Creates a Build and triggers it
def buildAloha(String project){
    sh "oc project ${project}"
    sh "oc start-build aloha"
    appDeploy()
}

// Tag the ImageStream from an original project to force a deployment
def deployAloha(String origProject, String project){
    sh "oc login -u openshift-dev -p devel"
    sh "oc project ${project}"
    sh "oc policy add-role-to-user system:image-puller system:serviceaccount:${project}:default -n ${origProject}"
    sh "oc tag ${origProject}/aloha:latest ${project}/aloha:latest"
    appDeploy()
}

// Deploy the project based on a existing ImageStream
def appDeploy(){
    sh "oc new-app aloha -l app=aloha,hystrix.enabled=true || echo 'Aplication already Exists'"
    sh "oc expose service aloha || echo 'Service already exposed'"
}

