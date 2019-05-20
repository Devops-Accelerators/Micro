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
    
    stage ('Check-secrets'){
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
    }	
	
    stage ('create war')
    {
    	mavenbuildexec "mvn build"
    }
    
     stage ('Create Docker Image')
    { 
	     echo 'creating an image'
	     docImg="${props['deploy.dockerhub']}/${props['deploy.microservice']}"
             dockerImage = dockerexec "${docImg}"
	    sh "sudo docker run -p 8083:8080 ${dockerImage}"
	    
    }    
    
    
    stage ('DAST') {
	    withCredentials([string(credentialsId:"zap", variable: 'zap')]){      

         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@54.72.96.92 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://99.81.179.32:8083/app/employee" || true'
      }
    }
	
}
