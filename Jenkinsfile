def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label) {

 node(label) {
	
	currentBuild.displayName = "1.${BUILD_NUMBER}"
	def GIT_COMMIT
  stage ('cloning the repository'){
      git 'https://github.com/tapansirol/AltoroJ.git'
  }
	
  stage('Gradle Build') {
       //def gradleHome = tool name : 'mygradle', type 'gradle',
        //sh "${gradleHome}/bin/gradle clean build"
        def path = tool name: 'gradle-4.7', type: 'gradle'
        sh "${path}/bin/gradle build"
   }
   
   stage('SonarQube analysis') {
//	   sleep 10
      def path = tool name: 'gradle-4.7', type: 'gradle'
      withSonarQubeEnv('sonar-server') {
        sh "${path}/bin/gradle --info -Dsonar.host.url=$SONAR_HOST_URL sonarqube"
    }
  }
	stage ("Appscan"){
		//appscan application: '84963f4f-0cf4-4262-9afe-3bd7c0ec3942', credentials: 'Credential for ASOC', failBuild: true, failureConditions: [failure_condition(failureType: 'high', threshold: 20)], name: '84963f4f-0cf4-4262-9afe-3bd7c0ec39421562', scanner: static_analyzer(hasOptions: false, target: '/var/jenkins_home/workspace/Altoro/build/libs/'), type: 'Static Analyzer'
	}
	
  stage('Publish Artificats to UCD'){
   step([$class: 'UCDeployPublisher',
        siteName: 'ucd-server',
        component: [
            $class: 'com.urbancode.jenkins.plugins.ucdeploy.VersionHelper$VersionBlock',
            componentName: 'AltoroComponent',
            createComponent: [
                $class: 'com.urbancode.jenkins.plugins.ucdeploy.ComponentHelper$CreateComponentBlock',
                componentTemplate: '',
                componentApplication: 'Altoro'
            ],
            delivery: [
                $class: 'com.urbancode.jenkins.plugins.ucdeploy.DeliveryHelper$Push',
                pushVersion: '1.${BUILD_NUMBER}',
                //baseDir: '/var/jenkins_home/workspace/JPetStore/target',
		 baseDir: '/home/jenkins/workspace/Altoro/build/libs/',
                fileIncludePatterns: '*.war',
                fileExcludePatterns: '',
               // pushProperties: 'jenkins.server=Jenkins-app\njenkins.reviewed=false',
                pushDescription: 'Pushed from Jenkins'
            ]
        ]
    ])
	  
		//sh 'env > env.txt'
		//readFile('env.txt').split("\r?\n").each {
		//println it
		//}
	echo "(*****)"
	  echo "Demo1234 ${AltoroComponent_VersionId}"
	  def newComponentVersionId = "${AltoroComponent_VersionId}"
	  echo "git commit ${GIT_COMMIT}"
	  step($class: 'UploadBuild', tenantId: "5ade13625558f2c6688d15ce", revision: "${GIT_COMMIT}", appName: "Altoro", requestor: "admin", id: "${newComponentVersionId}" )
	  echo "Demo123 ${newComponentVersionId}"
	sleep 25
	  step([$class: 'UCDeployPublisher',
		deploy: [ createSnapshot: [deployWithSnapshot: true, 
			 snapshotName: "1.${BUILD_NUMBER}"],
			 deployApp: 'Altoro', 
			 deployDesc: 'Requested from Jenkins', 
			 deployEnv: 'Altoro_Dev', 
			 deployOnlyChanged: false, 
			 deployProc: 'Deploy-Altoro', 
			 deployReqProps: '', 
			 deployVersions: "AltoroComponent:1.${BUILD_NUMBER}"], 
		siteName: 'ucd-server'])
     }
   }

}
