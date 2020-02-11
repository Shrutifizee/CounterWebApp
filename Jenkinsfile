#!groovy

/*********************************************************************
***** Description :: This template is used to setup Pipeline *****
* *** Author :: Ravindra Mittal ( ravindra.mittal@nagarro.com) *******
***** Date        :: 08/23/2017                                  *****
***** Revision    :: 1.0                                       *****
**********************************************************************/  

LABEL_EXPR='master'
JAVA_HOME='Java'
MAVEN_PATH='maven'
DEFAULT_RECIPIENTS='shruti.gupta@nagarro.com'
BUILDTOOL='MVN'
TARGET='main'
ARTIFACTORY_NAME='artifactory'
ARTIFACTORY_REPO='shruti'

def  funCodeCheckoutGit()
{ 
 echo  "\u2600 **********GIT Code Checkout Stage Begins*******"
checkout([$class: 'GitSCM', branches: [[name: "*/"]], doGenerateSubmoduleConfigurations: false, extensions: [], gitTool: 'Default', submoduleCfg: [], userRemoteConfigs: [[credentialsId: "24b76345-71e5-4762-94a7-4abf94f6070a", url: "https://github.com/Shrutifizee/CounterWebApp.git"]]])
} 


def fununitTestMvn()
{
echo  "\u2600 **********Running Unit test cases******************"
sh "${MAVEN_HOME}/bin/mvn test"
}

def funCodeBuildMvn()
{ 
echo  "\u2600 **********Build started******************" 
sh "${MAVEN_HOME}/bin/mvn install"
} 

def funSonarAnalysisMVN()
{ 
 echo  "\u2600 **********Sonar analysis started*****************" 
withSonarQubeEnv("Sonar") {
     sh "${MAVEN_HOME}/bin/mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar"
    }
} 

def funartifactoryUpload()
{
echo  "\u2600 **********Uploading to artifactory*****************" 
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
def funDockerCreateImage()
{
                           echo  "\u2600 **********CREATE DOCKER IMAGE*****************"
                           sh returnStdout: true, script: 'docker build -t docker.io/shrutifizeegupta/assignment:${BUILD_NUMBER} -f Dockerfile .'
} 
def funDockerPushImage()
{
             echo  "\u2600 **********PUSH DOCKER IMAGE to DTR*****************"
             sh returnStdout: true, script: 'docker push docker.io/shrutifizeegupta/assignment:${BUILD_NUMBER}'
}
def funDockerContainerStop(int p)
{
    echo  "\u2600 **********VERIFY IF RUNNING INSTANCE EXIST*****************"
             env.p=p;
    sh '''
    ContainerID=$(docker ps | grep $p | cut -d " " -f 1)
    if [  $ContainerID ]
    then
        docker stop $ContainerID
                           docker rm -f $ContainerID
    fi
    '''
}
def fundockercontRun()
{
             echo  "\u2600 **********STARTING DOCKER APPLICATION*****************"
             sh 'docker run --name devopssampleapplication_shrutigupta -d -p 7000:8080 docker.io/shrutifizeegupta/assignment:${BUILD_NUMBER}'
}
node("master")
{
             MAVEN_HOME = tool "maven"
             env.PATH = "${env.JAVA_HOME}/bin:${env.MAVEN_HOME}/bin:${env.ANT_HOME}/bin:${env.PATH}"
             try 
             {
                           stage 'Checkout'
                           funCodeCheckoutGit()
                           stage 'Build'
                           funCodeBuildMvn()
                           stage 'Unit Test'
                           fununitTestMvn()
                           stage 'Sonar Analysis'
                           funSonarAnalysisMVN()
						   stage 'Upload to Artifactory'
                           funartifactoryUpload()
			}
			catch (any)
			{
                           currentBuild.result = 'FAILURE'
                           throw any //rethrow exception to prevent the build from proceeding
			}
}
