pipeline {
	agent any
	tools {
    	maven 'local-mvn'
	}
    triggers {
        pollSCM '0 0 * * *'
    }
	stages {
    	stage("Checkout") {   
        	steps {               	 
            	git branch: 'main',  url: 'https://github.com/jdvalera/devops-project-4'          	 
        	}    
    	}
    	stage('Build') {
        	steps {
        	sh "mvn clean package"  	 
        	}
    	}
        stage('Ansible Deploy') {
            steps {
                ansiblePlaybook( 
                    playbook: 'tomcat_deploy.yaml',
                    inventory: 'inventory.ini', 
                    credentialsId: 'aws2') 
            }
        }
    }
}