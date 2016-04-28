node ("linux") {

    def local_path="gameoflife-web/target"
    def war="gameoflife.war"
    def target="/site/wwwroot/webapps"

    ensureMaven()

    stage "Checkout"
    git branch: 'azure-pipeline', url: 'https://github.com/harniman/game-of-life'

    stage "Build"

    sh "mvn clean package"
    
    stage "Deploy to Azure"

    withCredentials([[$class: 'UsernamePasswordMultiBinding', 
        credentialsId: 'azure-deployment-id', 
        passwordVariable: '_password', 
        usernameVariable: '_user']]) {

        sh "curl -T ${local_path}/${war} ftps://\"${env._user}\":${env._password}@${env.azureHost}${target}/"
    }

    
}

def ensureMaven() {
    env.PATH = "${tool 'maven-3.3'}/bin:${env.PATH}"
}





