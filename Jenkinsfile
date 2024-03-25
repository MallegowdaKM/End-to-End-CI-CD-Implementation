pipeline{
agent any
tools{
maven "maven_3.9.6"
}
parameters {
choice choices: ['master', 'main'], description: 'Select the branch name', name: 'BranchName'
}
options {
buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '1', numToKeepStr: '5')
timestamps()
}
stages{
stage("checkout"){
steps{
slack("STARTED")
cleanWs()
git branch: "${params.BranchName}", credentialsId: 'cbc6f3fd-5b34-4d4f-9006-d9297195ee64', url: 'https://github.com/MallegowdaKM/jenkins.git'
}
}
stage("build"){
steps{
sh "mvn clean package"
}
}
stage("executesonarqube"){
steps{
sh "mvn clean sonar:sonar"
}
}
stage("uploadartifactintonexus"){
steps{
sh "mvn clean deploy"
}
}
stage("deploymentappintitomcat"){
steps{
sshagent(['2e13cc75-48ba-4809-9b57-b8e6185ea387']){
sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@172.31.41.176:/opt/tomcat9/webapps/"
}
}
}
}//stages
post {
  aborted {
    // One or more steps need to be included within each condition's block.
    slack(currentBuild.result)
  }
  success {
    // One or more steps need to be included within each condition's block.
    slack(currentBuild.result)
  }
  failure {
    // One or more steps need to be included within each condition's block.
    slack(currentBuild.result)
  }
}
}//pipeline
  def slack(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    colorName= 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    colorName = 'GREEN'
    colorCode = '#00FF00'
  } else {
    colorName = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
