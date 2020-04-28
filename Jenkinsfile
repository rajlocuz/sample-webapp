pipeline {
  agent any
  tools {
    maven 'Maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
		echo "PATH = $(PATH)"
		echo "M2_HOME = $(M2_HOME)"
	   '''

      }
    }
    
   stage ('Check-Git-Secrets') {
      steps {
	sh 'rm trufflehogjson || true'
        sh 'rm trufflehog || true'
        sh 'docker run rajlocuz/trufflehog --json https://github.com/rajlocuz/webapp.git > trufflehogjson'
	sh 'docker run rajlocuz/trufflehog  https://github.com/rajlocuz/webapp.git > trufflehog'
        sh 'cat trufflehog'
	input 'Do you want to proceed?'
      }
    
    }

    stage ('Build') {
      steps {
        sh 'mvn clean package'
      }
    }
    
    stage ('Deploy-to-Tomcat') {
      steps {
        sshagent (['tomcat']) {
          sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@35.154.212.147:/opt/tomcat/webapps/webapp.war'
        }
      }
    }
  }
}
