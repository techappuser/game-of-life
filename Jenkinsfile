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
      stage 'Build Image'
	  {
		  //Packaging the image into a Docker image  
		   dockerImage = docker.build (appName.toLowerCase(), '.')
	  }
	  
	  stage('Push Image to ECR')
	  {
		  //Pushing image to ECR
		  docker.withRegistry (awsEcr + "/" + appName.toLowerCase(), "ecr:" + awsRegion + ":aws-credentials") {
			  sh 'ls -lart' 
			  dockerImage.push "latest"
	  }
	  }
	  stage('Deploy to ECS')
	  {
	  //Deploy image to ecs cluster in ECS
	  
			sh "/usr/local/bin/aws  ecs update-service --service ${ecsService}  --cluster ${ecsClusterName} --desired-count 0 >.amazon-ecs-service-update.json"
			timeout(time: 5, unit: 'MINUTES') {
				waitUntil {
					sh "/usr/local/bin/aws  ecs describe-services --service ${ecsService}  --cluster ${ecsClusterName}   > .amazon-ecs-service-status.json"
					def ecsServicesStatusAsJson = readFile(".amazon-ecs-service-status.json")
					def ecsServicesStatus = new groovy.json.JsonSlurper().parseText(ecsServicesStatusAsJson)
					def ecsServiceStatus = ecsServicesStatus.services[0]
					return ecsServiceStatus.get('runningCount') == 0 && ecsServiceStatus.get('status') == "ACTIVE"
				}
			}
			sh "/usr/local/bin/aws  ecs update-service --service ${ecsService}  --cluster ${ecsClusterName}  --desired-count 1 >.amazon-ecs-service-update.json"
			timeout(time: 5, unit: 'MINUTES') {
				waitUntil {
					sh "/usr/local/bin/aws  ecs describe-services --service ${ecsService}  --cluster ${ecsClusterName}  > .amazon-ecs-service-status.json"

					// parse `describe-services` output
					def ecsServicesStatusAsJson = readFile(".amazon-ecs-service-status.json")
					def ecsServicesStatus = new groovy.json.JsonSlurper().parseText(ecsServicesStatusAsJson)
				   // println "$ecsServicesStatus"
					def ecsServiceStatus = ecsServicesStatus.services[0]
					return ecsServiceStatus.get('runningCount') >= 1 && ecsServiceStatus.get('status') == "ACTIVE"
				}
			}
			
	  }
	  
	  stage('Test Application')
	  {
		  //Get task arn from task list
		sh "/usr/local/bin/aws ecs list-tasks --cluster ${ecsClusterName}  --service ${ecsService}  > .amazon-ecs-task-list.json"
		def ecsServiceTaskArn=sh(returnStdout: true, script: 'cat .amazon-ecs-task-list.json | jq ".taskArns[]"').trim()
		println "Task ARN : $ecsServiceTaskArn"
		
		//Get ENI Id from tasks
		sh "/usr/local/bin/aws ecs describe-tasks --task $ecsServiceTaskArn  > .amazon-ecs-task.json"
		def ecsServiceEniId=sh(returnStdout: true, script: "cat  .amazon-ecs-task.json | jq '.tasks[0].attachments[0].details[]' |grep  'eni-' | cut -d ':' -f2").trim()
		println "ENI Id : $ecsServiceEniId"
		
		// Get IP using eni
		sh "/usr/local/bin/aws ec2 describe-network-interfaces --network-interface-ids $ecsServiceEniId  > .amazon-ecs-nw-interface.json"
		def ecsTaskPublicIp=sh(returnStdout: true, script: "cat .amazon-ecs-nw-interface.json | jq '.NetworkInterfaces[].Association.PublicIp' | tr -d '\"' ").trim()
		println "PublicIp : $ecsTaskPublicIp"
		
		// Test public ip
        timeout(time: 2, unit: 'MINUTES') {
				waitUntil {
					try {
						sh "curl http://$ecsTaskPublicIp:8080"
						return true
					} catch (Exception e) {
						return false
					}
				}
			}
			echo "gameoflife#${env.BUILD_NUMBER} SUCCESSFULLY deployed to http://$ecsTaskPublicIp:8080"
		
	  }
	}
