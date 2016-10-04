#!groovy
import groovy.transform.Field
@Field def ARTIFACT_ID = ''
@Field def mesg = ''
if (env.BRANCH_NAME == 'develop') {
	stage 'Develop'
		node('maven') {
			checkout scm
			set_properties()
			mesg = sh(returnStdout: true, script: 'git log -n 1 --pretty=format:\'%s%n%n%b\'').trim()
			stage 'Build-Compile'
				build_java()			  
			stage 'Unit-Test'
				unit_test()		  
		  	stage 'Package-Upload' 	  
		  		package_upload()		  	
		}
	stage 'Deploy_HD-WWW-DEV'
		node('maven') {
		  		deploy_gcp()
		  	}
	stage 'Other-Test'
		  		parallel 'sonar':{
		  				node('maven') {
		  					echo 'Running sonar test'
		  					sonar_test()
		  				}
		  			}, 'performance':{
		  				node('k8s') {
		  					
		  					if (mesg =~ 'run-perf-test') {
								echo "Starting load test"
								perf_test()
							} else {
								echo "skipping load test"
							}
		  				}

		  				}, 'security': {
		  					node('k8s') {
		  						
		  						if (mesg =~ '-run-security-scan') {
									echo 'Starting security scan'
									security_test()

								} else {
									echo "skipping security scan"
								}
		  					}
		  				}
} else if (env.BRANCH_NAME == 'stage' || env.BRANCH_NAME == 'master') {
		stage 'Stage'
		node('maven') {
			checkout scm
			set_properties()

			stage 'Build-Compile'
				build_java()
			stage 'Deploy_HD-WWW-STAGE'
		  		deploy_gcp()
		}
} else {
	//feature

}
def set_properties() {
	def TMPDIR = '/tmp'
	sh 'mvn clean antrun:run@generate-properties -U'
	def d = [test: 'Default', something: 'Default', other: 'Default']
	def BUILD_PROPERTIES = readProperties(defaults: d, file: TMPDIR + '/spring-boot-sample-simple.properties', text: 'other=Override')
	ARTIFACT_ID = BUILD_PROPERTIES['project.artifactId']
}
	
def build_java() {
	
	sh 'mvn clean compile'	
}

def unit_test() {
	sh 'mvn test -B verify'
}

def package_upload() {
	sh 'mvn install deploy -Dmaven.test.skip=true'
}

def version() {
	def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
	matcher ? matcher[0][1] : null
}

def deploy_gcp() {
	//- if test -z "$TMPDIR" ; then export TMPDIR="/tmp" ; fi
	def TMPDIR = '/tmp'
	def PROJECT = sh(returnStdout: true, script: '/usr/share/google/get_metadata_value project-id').trim()
	
	def d = [test: 'Default', something: 'Default', other: 'Default']
	def BUILD_PROPERTIES = readProperties(defaults: d, file: TMPDIR + '/spring-boot-sample-simple.properties', text: 'other=Override')
	def GROUP_ID= BUILD_PROPERTIES['project.groupId']
	def VERSION= BUILD_PROPERTIES['project.version']
	//def CDM_TMPL_URL = 'https://storage.googleapis.com/'+PROJECT+'-bootstrap/cdm-template-version.txt'
	//def httpRequestObject= httpRequest CDM_TMPL_URL
	//def CDM_TEMPLATE_VERSION = httpRequestObject.content
	def CDM_TEMPLATE_VERSION = 'v2.0.10'
	//hammer --show-version
	def CONSUL_HEALTH_URI='catalog/admin/health'
	def HEALTH_CHECK_URI="/" + CONSUL_HEALTH_URI
	def HEALTH_CHECK_PORT=8080

	echo PROJECT
	echo VERSION
	echo ARTIFACT_ID
	echo GROUP_ID
	echo CDM_TEMPLATE_VERSION

	//sh 'gsutil rsync -d -r bootstrap/ gs://'+PROJECT+'-artifacts/releases/'+GROUP_ID+'/'+ARTIFACT_ID+'/'+VERSION+'/data-load/bootstrap/deploy_autohealing.sh -d olt-app-'+ARTIFACT_ID+'-ah -t ./patterns/autohealing.jinja -u '+HEALTH_CHECK_URI+' -p '+HEALTH_CHECK_PORT
	sh 'cp hammer-properties.yaml hammer-properties.yaml.orig'
	sh 'sed -i.bak \'s/%VERSION%/'+VERSION+'/g\' hammer-properties.yaml'
	sh 'sed -i.bak \'s/%CDM_TMPL_VER%/'+CDM_TEMPLATE_VERSION+'/g\' hammer-properties.yaml'
	sh 'sed -i.bak \'s/%PROJECT%/'+PROJECT+'/g\' hammer-properties.yaml'
	sh 'sed -i.bak \'s/%ARTIFACT_ID%/'+ARTIFACT_ID+'/g\' hammer-properties.yaml'
	sh 'sed -i.bak \'s/%GROUP_ID%/'+GROUP_ID+'/g\' hammer-properties.yaml'
	sh 'cat hammer-properties.yaml ; echo'
	/*
	hammer -p ${PROJECT} --propertiesFile "hammer-properties.yaml" -d --deploymentType ActiveRotate_v5 --noPause
	*/
}

def sonar_test() {
	//sh 'mvn clean verify sonar:sonar'
	//sh 'mvn clean verify jacoco:check'
}

def security_test() {	
	//sh 'fortify-scan -a '+ARTIFACT_ID+' -d someone@homedepot.com'	
}

def perf_test() {
	
		/*
		sh '. cicd/perf/.envrc'
    	sh 'cicd/perf/uploadPerf.sh --no-prompt'
    	sh 'git clone git@git.hdtechlab.com:cloud/perf-test-kubernetes.git'
    	sh 'cd perf-test-kubernetes'
    	sh 'set -x'
    	sh '. run-perf-test.sh'
    	*/
	
}






