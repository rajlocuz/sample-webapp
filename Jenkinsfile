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
          sh 'rm trufflehog || true'
	  sh 'docker run rajlocuz/trufflehog  https://github.com/rajlocuz/sample-webapp.git > trufflehog'
          sh 'cat trufflehog'
	  input 'Do you want to proceed?'
      }
    
    }

   stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/rajlocuz/sample-webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
        
      }
    }

   stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }

    stage ('Build') {
      steps {
        sh 'mvn clean package'
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
            sh 'mvn failsafe:integration-test'
	      input 'Do you wish to continue?'
          }
        }
     }
     post {
       always {
         junit allowEmptyResults: true, testResults: 'target/surefire-reports/Test-*.xml'
       }

       failure {
         mail to: 'raj.singh@locuz.com', subject: 'The Pipeline failed :(', body:'The Pipeline failed :('
       }
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
