pipeline
{
    agent any
	tools
	{
		maven 'maven'
	}
	options
    {
        timestamps()
        timeout(time: 1, unit: 'HOURS')
        skipDefaultCheckout()
        buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))
	      disableConcurrentBuilds()
    }
    stages
    {
	    stage ('checkout')
		{
			steps
			{
				checkout scm
			}
		}
		stage ('Build')
		{
			steps
			{
				sh "mvn clean install"
			}
		}
		stage ('Unit Testing')
		{
			steps
			{
				sh "mvn test"
			}
		}
		stage ('Sonar Analysis')
		{
			steps
			{
				withSonarQubeEnv("Sonar") 
				{
					sh "mvn sonar:sonar"
				}
			}
		}
		stage ('Upload to Artifactory')
		{
			steps
			{
				def server = Artifactory.server 'artifactory'
     				def buildInfo = Artifactory.newBuildInfo()
      				buildInfo.env.capture = true
      				buildInfo.env.collect()
      				def rtMaven = Artifactory.newMavenBuild()
      				rtMaven.tool = 'maven'
      				rtMaven.deployer releaseRepo:'shruti', snapshotRepo:'shruti', server: server
    				rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
    				buildInfo.retention maxBuilds: 10, maxDays: 7, deleteBuildArtifacts: true
             			server.publishBuildInfo buildInfo
			}
		}
		stage ('Docker Image')
		{
			steps
			{
				sh returnStdout: true, script: 'docker build -t docker.io/shrutifizeegupta/assignment:${BUILD_NUMBER} -f Dockerfile .'
			}
		}
		stage ('Push to DTR')
	    {
		    steps
		    {
		    	sh returnStdout: true, script: 'docker push docker.io/shrutifizeegupta/assignment:${BUILD_NUMBER}'
		    }
	    }
        stage ('Stop Running container')
    	{
	        steps
	        {
	            sh '''
                    ContainerID=$(docker ps | grep 7000 | cut -d " " -f 1)
                    if [  $ContainerID ]
                    then
                        docker stop $ContainerID
                        docker rm -f $ContainerID
                    fi
                '''
	        }
	    }

		stage ('Docker deployment')
		{
		    steps
		    {
		        sh 'docker run --name devopssampleapplication_shrutigupta -d -p 7000:8080 docker.io/shrutifizeegupta/assignment:${BUILD_NUMBER}'
		    }
		}
	}
	post 
	{
        always 
		{
			emailext attachmentsPattern: 'report.html', body: '${JELLY_SCRIPT,template="health"}', mimeType: 'text/html', recipientProviders: [[$class: 'RequesterRecipientProvider']], replyTo: 'shruti.gupta@nagarro.com', subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: 'shruti.gupta@nagarro.com'
        }
    }
}
