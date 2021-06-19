//Picking a build agent labeled "ec2" to run pipeline on
node (){
  def buildNumber=params.APP_MAJOR_VERSION + "." + env.BUILD_NUMBER
  stage 'Preflight'
   if(params.appMajorVersion)
	{
		println 'appMajorVersion : ' + params.appMajorVersion
	}
	else
	{
		println 'appMajorVersion is empty'
		System.exit(0)
	} 
  currentBuild.displayName = "${buildNumber}"
  stage 'Pull from SCM'  
  //Passing the pipeline the ID of my GitHub credentials and specifying the repo for my app
  git credentialsId: '32f2c3c2-c19e-431a-b421-a4376fce1186', url: 'https://github.com/techappuser/game-of-life.git'
  stage 'Test Code'  
 /*
  sh 'mvn install'
 
  stage 'Build App' 
  //Running the maven build and archiving the war
  sh 'mvn install'
  archive 'target/*.war'
  
  stage 'Build Image'
  //Packaging the image into a Docker image
  def pkg = docker.build ('lavaliere/game-of-life', '.')

  
  stage 'Push Image to ECR'
  //Pushing the packaged app in image into DockerHub
  docker.withRegistry ('https://206748326815.dkr.ecr.us-east-2.amazonaws.com/lavaliere/game-of-life', 'ecr:us-east-2:aws-credentials') {
      sh 'ls -lart' 
      pkg.push 'docker-demo'
  }
  
  stage 'Deploy to ECS'
  //Deploy image to production in ECS
        sh "aws ecs update-service --service production-deploy-game  --cluster production --desired-count 0"
        timeout(time: 5, unit: 'MINUTES') {
            waitUntil {
                sh "aws ecs describe-services --service production-deploy-game  --cluster production   > .amazon-ecs-service-status.json"

                // parse `describe-services` output
                def ecsServicesStatusAsJson = readFile(".amazon-ecs-service-status.json")
                def ecsServicesStatus = new groovy.json.JsonSlurper().parseText(ecsServicesStatusAsJson)
                println "$ecsServicesStatus"
                def ecsServiceStatus = ecsServicesStatus.services[0]
                return ecsServiceStatus.get('runningCount') == 0 && ecsServiceStatus.get('status') == "ACTIVE"
            }
        }
        sh "aws ecs update-service --service production-deploy-game  --cluster production  --desired-count 1"
        timeout(time: 5, unit: 'MINUTES') {
            waitUntil {
                sh "aws ecs describe-services --service production-deploy-game  --cluster production  > .amazon-ecs-service-status.json"

                // parse `describe-services` output
                def ecsServicesStatusAsJson = readFile(".amazon-ecs-service-status.json")
                def ecsServicesStatus = new groovy.json.JsonSlurper().parseText(ecsServicesStatusAsJson)
                println "$ecsServicesStatus"
                def ecsServiceStatus = ecsServicesStatus.services[0]
                return ecsServiceStatus.get('runningCount') == 0 && ecsServiceStatus.get('status') == "ACTIVE"
            }
        }
        timeout(time: 5, unit: 'MINUTES') {
            waitUntil {
                try {
                    sh "curl http://52.202.249.4:80"
                    return true
                } catch (Exception e) {
                    return false
                }
            }
        }
        echo "gameoflife#${env.BUILD_NUMBER} SUCCESSFULLY deployed to http://52.202.249.4:80"
    
  */
}
