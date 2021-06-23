	node (){
	  def buildNumber=params.appMajorVersion + "." + env.BUILD_NUMBER
	  def appName=params.appName
	  def appMajorVersion=params.appMajorVersion
	  def ecsClusterName=params.ecsClusterName
	  def ecsTaskDefinition=params.ecsTaskDefinition
	  def ecsService=params.ecsService
	  def awsEcr=params.awsEcr
	  def awsRegion=params.awsRegion
	  def dockerImage
	  
	  stage('Preflight')
	  {
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
		
		//Update Build Displayname
		currentBuild.displayName = "${appName}" + " - " +"${buildNumber}"
	  }
	  
	  stage('Pull from SCM') 
	  {  
		  //Passing the pipeline the ID of my GitHub credentials and specifying the repo for my app
		  git credentialsId: 'git-access', url: 'https://github.com/techappuser/game-of-life.git'
	  }
	  stage('Code Analysis')  
	  {
		println 'Code Scan Stage'
	  }
		stage('Build App') 
	  {
		  //Running the maven build and archiving the war
		  sh 'mvn install'
		  archive 'target/*.war'
	  }  
      stage('Build Image')
	  {
		  //Packaging the image into a Docker image  
		   dockerImage = docker.build (appName.toLowerCase(), '.')
	  }
	  
	  stage('Push Image to ECR')
	  {
		  //Pushing image to ECR
		  docker.withRegistry ("https://" + awsEcr + "/" + appName.toLowerCase(), "ecr:" + awsRegion + ":aws-credentials") 
		   {
			  sh 'ls -lart' 
			  dockerImage.push "${buildNumber}"
			  
			}
	  }
	  stage('Deploy to ECS')
	  {   
	     
		  //Get current task definition json
		  sh("/usr/local/bin/aws ecs describe-task-definition --task-definition $ecsTaskDefinition --region $awsRegion --output json > $ecsTaskDefinition'.json'")
		  
		  // Create a new task definition for this build
		  sh("cat $ecsTaskDefinition'.json' | jq '.taskDefinition.containerDefinitions[].image=\"$awsEcr:$buildNumber\"' | jq '.taskDefinition|{networkMode : .networkMode, family: .family, placementConstraints: .placementConstraints, cpu : .cpu, executionRoleArn: .executionRoleArn,  volumes: .volumes, memory : .memory, requiresCompatibilities : .requiresCompatibilities,  containerDefinitions: .containerDefinitions}' > $ecsTaskDefinition'_'$buildNumber'.json'")
		  
		  //Register new task definition
		  sh("/usr/local/bin/aws ecs register-task-definition --family $ecsTaskDefinition --region $awsRegion --requires-compatibilities FARGATE --cli-input-json file://$ecsTaskDefinition'_'$buildNumber'.json'")
		  
		  // Update the service with the new task definition and desired count
		  taskRevision=sh(returnStdout: true, script: "/usr/local/bin/aws ecs describe-task-definition --task-definition $ecsTaskDefinition --region $awsRegion | jq .taskDefinition.revision").trim()
			
		  println 'New Task Revision : ' +  taskRevision
			
		  //Get Desire Count
		  cntDesired=sh(returnStdout: true, script: "/usr/local/bin/aws ecs describe-services --cluster $ecsClusterName --services $ecsService --region $awsRegion | jq .services[].desiredCount").trim()
		  
		  if (cntDesired == "0")
		  {
			  println 'Current Desire count is 0. So Setting it to 1 '
			  cntDesired="1"
		  }
		  
		  //Update service
		  sh("/usr/local/bin/aws ecs update-service --cluster $ecsClusterName --service $ecsService --task-definition $ecsTaskDefinition:$taskRevision --desired-count $cntDesired")
			
		timeout(time: 5, unit: 'MINUTES') {
			waitUntil {
				sh "/usr/local/bin/aws  ecs describe-services --service ${ecsService}  --cluster ${ecsClusterName}  > .amazon-ecs-service-status.json"
				def ecsServicesStatusAsJson = readFile(".amazon-ecs-service-status.json")
				def ecsServicesStatus = new groovy.json.JsonSlurper().parseText(ecsServicesStatusAsJson)
				def ecsServiceStatus = ecsServicesStatus.services[0]
				return ecsServiceStatus.get('runningCount') >= 1 && ecsServiceStatus.get('status') == "ACTIVE"
			}
		}
			
	  }  
	  stage('Test Application')
	  {
		 
        timeout(time: 2, unit: 'MINUTES') {
				waitUntil {
					try {
						sh "curl $loadBalancerUrl:8080"
						return true
					} catch (Exception e) {
						return false
					}
				}
			}
			echo "gameoflife#${env.BUILD_NUMBER} SUCCESSFULLY deployed to http://$ecsTaskPublicIp:8080"
		
	  }
	}
