node ("linux") {

    def local_path="gameoflife-web/target"
    def war="gameoflife.war"
    def azureHost="waws-prod-db3-047.ftp.azurewebsites.windows.net"

    ensureMaven()
    git branch: 'azure-pipeline', url: 'https://github.com/harniman/game-of-life'
    sh "mvn clean package"
    
    
    def target="/site/wwwroot/webapps"
    
    deployToAzure(azureHost, local_path, war, target)
    
    
    
}

def ensureMaven() {
    env.PATH = "${tool 'maven-3.3'}/bin:${env.PATH}"
}

def deployToAzure(host, local_path, file_name, remote_path) {
withCredentials([[$class: 'UsernamePasswordMultiBinding', 
        credentialsId: 'azure-deployment-id', 
        passwordVariable: '_password', 
        usernameVariable: '_user']]) {

  echo env._user

  writeFile file: 'ftp_script.sh', text: '''#!/bin/sh
      HOST=\'''' + host + '''\'

      ftp -p -n $HOST <<END_SCRIPT
      quote USER "$_user"
      quote PASS "$_password"
      pwd
      cd "''' + remote_path + '''"
      pwd
      lcd ''' + local_path + '''
      binary
      put ''' + file_name + '''
      bye
      END_SCRIPT
      exit 0'''
  echo 'Generated temporary FTP script'
  sh "set; cat ftp_script.sh"
  sh 'chmod +x ftp_script.sh'
  echo 'Executing FTP upload...'
  sh './ftp_script.sh'
  echo 'Removing temporary FTP script'
  sh 'rm ./ftp_script.sh'
}


}



