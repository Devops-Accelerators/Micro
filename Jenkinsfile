@Library('my_shared_library')_

def workspace;
def branch;
def dockerImage;
def props='';
def microserviceName;
def port;
def docImg;
def repoName;
def credentials = 'docker-credentials';

node {
    stage('Checkout Code')
    {
	checkout scm
	workspace = pwd() 
	     sh "ls -lat"
	props = readProperties  file: """deploy.properties"""   
    }
    
   /* stage ('Check-secrets'){
    	sh """
	rm trufflehog | true
	docker run gesellix/trufflehog --json ${props['deploy.gitURL']} > trufflehog
	cat trufflehog	
	"""
    }

    stage ('Source Composition Analysis') {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/Devops-Accelerators/Micro/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
    }*/	
	
    stage ('create war')
    {
    	mavenbuildexec "mvn build"
    }
    
     stage ('Create Docker Image')
    { 
	    app = docker.build("devopsaccelerator/micro1:1-${BUILD_NUMBER}")
    }
	
	stage('Push Image') {
		docker.withRegistry('https://registry.hub.docker.com','docker-credentials') {
			app.push("1-${BUILD_NUMBER}")
			app.push("latest")
		}
	    
    }    
	stage ('container') {
		sh "sudo docker run -p 8084:8080 -d devopsaccelerator/micro1"
	}
    
    stage ('DAST') {
	   
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@54.72.96.92 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://99.81.179.32:8084/app/employee" || true'
      
    }
	
}
