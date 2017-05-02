#!groovy

//input message: '', parameters: [[$class: 'GitParameterDefinition', branchFilter: '.*', defaultValue: '*/master', name: 'tag', tagFilter: '*', type: 'PT_BRANCH_TAG', Description: '']]
//def res = input (message: '', parameters: [[$class: 'TextParameterDefinition', id: 'res', name: 'tag', defaultValue: '*/master', Description: '']])
def tag="*/master"
def LABEL_MASTER="default_master"
def LABEL_LOWER="default_lower"
def LABEL_UPPER="default_lower"
def CREDS="6c123f4b5bb745f19c2a29391a0692c8419acb2c"
def URL='https://github.com/CBIIT/pipeline-as-code-demo.git'
def TIME1=5
def TIME2=10

def mvn(args) {
  withEnv(["JAVA_HOME=${tool 'java8'}", "M2_HOME=${tool 'M3'}"]) {
    sh "${M2_HOME}/bin/mvn ${args}"
  }
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

//print parameters
echo ("tag=${tag}")

//run on master
node(LABEL_MASTER)
{
  //dump env
  sh 'env > env.txt'
  s = readFile('env.txt').split("\r?\n")
  for(i=0; i <s.length; i++) {
      println s[i]
  }
}

//checkpoint and clean workspace
node {
  stash('Before cleanup')
  step([$class: 'WsCleanup'])
}

//run on master
node {
    //checkout
    checkout([$class: 'GitSCM', branches: [[name: tag]], doGenerateSubmoduleConfigurations: false
      , extensions: [], gitTool: 'Default', submoduleCfg: [], userRemoteConfigs: [[credentialsId: CREDS
      , url: URL]]])
    stash('checkout')      
}

//run on slave
node (LABEL_LOWER) {
    stage (name: 'Dev') {
      unstash('checkout')
      //build
      mvn 'clean package'
      //deploy
      dir('target') {stash name: 'war', includes: 'x.war'}
    }
}

//run on slave
node(LABEL_LOWER) {
  stage (name: 'QA') {
    parallel(
      longerTests: {
        runTests(TIME1, LABEL_LOWER)
      }, quickerTests: {
        runTests(TIME2, LABEL_LOWER)
      }
    )
  }
}

//run on slave
node (LABEL_UPPER) {
  stage (name: 'Stage') {
    deploy 'staging'
  }
}

//run on master
node
{
  stage( name: 'Notify approver')
  {
    mail bcc: '', body: "${JOB_URL}", cc: '', from: '', replyTo: '', subject: "Request deploy to production (Build ${BUILD_ID})", to: 'hartmanph@nih.gov'
  }
  // stage( name: 'Wait for approval')
	// {
    // timeout(time: 1, unit: 'HOURS'){
      // input message: 'Deploy to prod?', submitter: 'hartmanp'
    // }
	// }
}

// node(LABEL_MASTER) {
  // stash('Before production')
// }

// node (LABEL_UPPER){
  // stage name: 'Production' {
    // echo 'Production server looks to be alive'
    // deploy 'production'
    // echo "Deployed to production"
  // }
// }

