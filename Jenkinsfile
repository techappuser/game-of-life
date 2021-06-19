//Picking a build agent labeled "ec2" to run pipeline on
node (){
  def buildNumber=params.appMajorVersion + "." + env.BUILD_NUMBER
  def appName=params.appName
  def appMajorVersion=params.appMajorVersion
  def ecsClusterName=params.ecsClusterName
  def ecsTaskDefinition=params.ecsTaskDefinition
  def ecsService=params.ecsService
  def awsEcr=params.awsEcr
  def awsRegion=params.awsRegion
  
  stage 'Preflight'
   if(appName)
	{
		println 'appName : ' + appName
	}
	else
	{
		error('appName is empty')
	} 
   if(appMajorVersion)
	{
		println 'appMajorVersion : ' + appMajorVersion
	}
	else
	{
		error('appMajorVersion is empty')
	}
	if(ecsClusterName)
	{
		println 'ecsClusterName : ' + ecsClusterName
	}
	else
	{
		error('ecsClusterName is empty')
	}
	if(ecsTaskDefinition)
	{
		println 'ecsTaskDefinition : ' + ecsTaskDefinition
	}
	else
	{
		error('ecsTaskDefinition is empty')
	}
	if(ecsService)
	{
		println 'ecsService : ' + ecsService
	}
	else
	{
		error('ecsService is empty')
	}
	if(awsEcr)
	{
		println 'ecsService : ' + ecsService
	}
	else
	{
		error('ecsService is empty')
	}
	
	
  currentBuild.displayName = "${buildNumber}"
  stage 'Pull from SCM'  
  //Passing the pipeline the ID of my GitHub credentials and specifying the repo for my app
  git credentialsId: '32f2c3c2-c19e-431a-b421-a4376fce1186', url: 'https://github.com/techappuser/game-of-life.git'
  stage 'Test Code'  
 
  sh 'mvn install'
 
  stage 'Build App' 
  //Running the maven build and archiving the war
  sh 'mvn install'
  archive 'target/*.war'
  
  stage 'Build Image'
  //Packaging the image into a Docker image
  def pkg = docker.build (appName, '.')

  
  stage 'Push Image to ECR'
  //Pushing the packaged app in image into DockerHub
  docker.withRegistry ("${awsEcr}" + "/" + "${appName}", "ecr:" + "${awsRegion}" + ":aws-credentials") {
      sh 'ls -lart' 
      pkg.push "${buildNumber}"
  }
  /*
  stage 'Deploy to ECS'
  //Deploy image to ecs cluster in ECS
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
