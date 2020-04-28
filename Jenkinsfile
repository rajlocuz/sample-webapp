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
	//sshagent (['Jenkins-key']) {
	  //sh 'rm trufflehogjson || true'
          sh 'rm trufflehog || true'
          //sh 'docker run rajlocuz/trufflehog --json https://github.com/rajlocuz/webapp.git > trufflehogjson'
	  sh 'docker run rajlocuz/trufflehog --json  https://github.com/rajlocuz/sample-webapp.git > trufflehog'
          sh 'cat trufflehog'
	  input 'Do you want to proceed?'
	//}
      }
    
    }

    stage ('Build') {
      steps {
        sh 'mvn clean package -DskipTests=true'
      }
    }
    stage ('Tests') {
      failFast true	
      parallel {
    	stage('Unit Tests') {
      	  steps {
            sh 'mvn surefire:test'
          } 
        }

        stage('Integration Tests') {
          steps {
            sh 'mvn failsafe:integration-test || true'
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
	      input 'Do you wish to continue?'
            }
          }
        }
     }
/*     post {
       always {
         junit 'target/surefire-reports/TEST-*.xml'
	 //input 'Do you want to proceed?'
       }

       failure {
         mail to: 'raj.singh@locuz.com', subject: 'The Pipeline failed :(', body:'The Pipeline failed :('
         //input 'Do you want to proceed?'
       }
    }*/
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
