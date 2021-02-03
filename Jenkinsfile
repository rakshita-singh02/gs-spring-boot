pipeline {
     
  agent {
    kubernetes {
      label 'SpringBootRestApp'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  containers:
  - name: gradle
    image: gradle:3.5-jdk8-alpine
    command:
    - cat
    tty: true
"""
}
    }
  
stages {
	stage('Build & Unit Test') {
		steps {
			container('gradle') {
				withMaven(maven: 'MAVEN-3.6.3') {
					withSonarQubeEnv(installationName: 'Sonarqube') {
						echo 'I am executing unit test'
						// sh 'for i in ESBAuditClient ESBAuditLog ESBErrorTranslator TaxESB FraudESB FulfillmentESB PaymentESB ESBRadial ESBAutomatedQueueRetry AlertESB OrderReconESB;do gradle --no-daemon -p ${i} clean build;done'
						//sh 'mvn -f sample-java-app/pom.xml clean package'
						//sh 'mvn -X javadoc:javadoc -o'
						sh 'cd complete'
						sh './build.gradle'
					}
				}
			}
		}
	}
	
   }
}
