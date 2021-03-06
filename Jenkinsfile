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
            input 'Have you checked all vulnerability? Do you wish to deploy on Staging/QA Env.?'
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
 
    stage ('Deploy-to-Staging/QA') {
      steps {
        sshagent (['tomcat']) {
          sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@15.206.150.148:/opt/tomcat/webapps/webapp.war'
        }
      }
    }

    stage ('DAST') {
      steps {
        sshagent(['zap']) {
          sh 'ssh -o  StrictHostKeyChecking=no ec2-user@13.127.32.73 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://15.206.150.148:8080/webapp/" || true'
           
        }
      }
    }

    stage ('Deploy-to-Prod') {
      steps {
        sshagent (['tomcat']) {
          sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@35.154.212.147:/opt/tomcat/webapps/webapp.war'
          input 'Have you checked all vulnerability? Do you wish to deploy on Prod. Env.?'
        }
      }
    }

  }
}

