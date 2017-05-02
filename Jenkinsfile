#!groovy
/**
  * test
  */
//input message: '', parameters: [[$class: 'GitParameterDefinition', branchFilter: '.*', defaultValue: '*/master', name: 'tag', tagFilter: '*', type: 'PT_BRANCH_TAG', Description: '']]
//def res = input (message: '', parameters: [[$class: 'TextParameterDefinition', id: 'res', name: 'tag', defaultValue: '*/master', Description: '']])
def res=[name:"*/master"]
def LABEL_LOWER="slave-pool-1"
def LABEL_UPPER="slave-pool-2"
def CREDS="  6d6e8a0f-c9f9-4177-8e8a-bcbf58de8bf6"
def URL='https://github.com/jacksonm111-org/pipeline-as-code-demo.git'
def TIME1=5
def TIME2=10

def mvn(args) {
    sh "${tool 'M3'}/bin/mvn ${args}"
}

def runTests(duration, label) {
    node(label) {
      sh "ls -la"
        sh "sleep ${duration}"
        }
    }

def deploy(id) {
    unstash 'war'
    sh "cp x.war /tmp/${id}.war"
}

def undeploy(id) {
    sh "rm /tmp/${id}.war"
}

//start
node {
    sh 'env > env.txt'
    s = readFile('env.txt').split("\r?\n")
    for(i=0; i <s.length; i++) {
        println s[i]
    }
}

try {
    checkpoint('Before Dev')
} catch (NoSuchMethodError _) {
    echo 'Checkpoint feature available in CloudBees Jenkins Enterprise.'
}
echo '10'
def tag = res['name']
echo '11'
echo ("tag=${tag}")
echo '13'

stage 'Dev'
node (LABEL_LOWER) {
    checkout([$class: 'GitSCM', branches: [[name: "ref/heads/${tag}"]], doGenerateSubmoduleConfigurations: false
      , extensions: [], gitTool: 'Default', submoduleCfg: [], userRemoteConfigs: [[credentialsId: CREDS
      , url: URL]]])    
    mvn 'clean package'
    dir('target') {stash name: 'war', includes: 'x.war'}
}
echo '21'

stage 'QA'
node(LABEL_LOWER) {
  parallel(
    longerTests: {
      runTests(TIME1, LABEL_LOWER)
    }, quickerTests: {
      runTests(TIME2, LABEL_LOWER)
    }
  )
}

stage name: 'Staging', concurrency: 1
node (LABEL_UPPER) {
  deploy 'staging'
}

stage 'Approval'
node {
  mail bcc: '', body: "${JOB_URL}", cc: '', from: '', replyTo: '', subject: 'Request deploy to production', to: 'hartmanph@nih.gov'
  timeout(time: 1, unit: 'HOURS'){
    input message: "Does staging look good?", submitter: "hartmanp"
  }
}

try {
    checkpoint('Before production')
} catch (NoSuchMethodError _) {
    echo 'Checkpoint feature available in CloudBees Jenkins Enterprise.'
}

stage name: 'Production', concurrency: 1
node (LABEL_UPPER){
    echo 'Production server looks to be alive'
    deploy 'production'
    echo "Deployed to production"
}

