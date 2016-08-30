node {
//    stage 'Git checkout'
//    echo 'Checking out git repository'
//    git url: 'https://github.com/tariq-islam/aloha'

    echo 'Building project'

    stage 'Build image and deploy in Dev'
    echo 'Building docker image and deploying to Dev'
    buildAloha('helloworld-msa-dev')

//    stage 'Automated tests'
//    echo 'This stage simulates automated tests'
//    sh "${mvnHome}/bin/mvn -B -Dmaven.test.failure.ignore verify"

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
    projectSet(project)
    sh "oc new-build https://github.com/tariq-islam/bluegreen || echo 'Build exists'"
    sh "oc start-build bluegreen --follow"
    appDeploy()
}

// Tag the ImageStream from an original project to force a deployment
def deployAloha(String origProject, String project){
    projectSet(project)
    sh "oc policy add-role-to-user system:image-puller system:serviceaccount:${project}:default -n ${origProject}"
    sh "oc tag ${origProject}/bluegreen:latest ${project}/bluegreen:latest"
    appDeploy()
}

// Login and set the project
def projectSet(String project){
    //Use a credential called openshift-dev
    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'openshift-dev', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
        sh "oc login --insecure-skip-tls-verify=true -u $env.USERNAME -p $env.PASSWORD https://$OPENSHIFT:8443"
    }
    sh "oc new-project ${project} || echo 'Project exists'"
    sh "oc project ${project}"
}

// Deploy the project based on a existing ImageStream
def appDeploy(){
    sh "oc new-app bluegreen -l app=bluegreen || echo 'Application already Exists'"
    sh "oc expose service bluegreen || echo 'Service already exposed'"
//    sh 'oc patch dc/aloha -p \'{"spec":{"template":{"spec":{"containers":[{"name":"aloha","ports":[{"containerPort": 8778,"name":"jolokia"}]}]}}}}\''
//    sh 'oc patch dc/aloha -p \'{"spec":{"template":{"spec":{"containers":[{"name":"aloha","readinessProbe":{"httpGet":{"path":"/api/health","port":8080}}}]}}}}\''
}
