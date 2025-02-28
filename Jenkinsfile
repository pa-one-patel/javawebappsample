import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=/subscriptions/d8ab895d-71f4-4103-9b6e-952ffa536fde',
        'AZURE_TENANT_ID=00adb040b-ca22-4ca6-9447-ab7b049a22ff']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'QuickstartJenkins-rg'
      def webAppName = 'mySimpleApp2'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'MqXdih~m1pCZA0_dRwFdp_yQBJo1vFirUm', usernameVariable: 'b686b976-9902-436a-afa5-6e0fb55238bf')]) {
       sh '''
          az login --service-principal -u b686b976-9902-436a-afa5-6e0fb55238bf -p MqXdih~m1pCZA0_dRwFdp_yQBJo1vFirUm -t 0adb040b-ca22-4ca6-9447-ab7b049a22ff
          az account set -s d8ab895d-71f4-4103-9b6e-952ffa536fde
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
