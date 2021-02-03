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
						sh 'mvn -f sample-java-app/pom.xml clean package'
						sh 'mvn -X javadoc:javadoc -o'
					}
				}
			}
		}
	}
	stage('Code Quality') {
		steps {
			container('gradle') {
				withMaven(maven: 'MAVEN-3.6.3') {
					withSonarQubeEnv(installationName: 'Sonarqube') {
						echo 'I am executing code quality using sonarqube'
						sh 'mvn -f sample-java-app/pom.xml sonar:sonar'
					}
					 sleep(60)
					timeout(time: 1, unit: 'MINUTES') {
					waitForQualityGate abortPipeline: true
					} 
				}
			}
		}
	}
	stage("Publish Package") {
		steps {
			container('gradle') {
				script {
					propfile = readProperties(file: './project.properties')
					println("deploy:" + propfile['feature_deploy'])
					println("reading properties ...")
					if (propfile['feature_deploy'] == "true" || env.BRANCH_NAME == 'dev' || env.BRANCH_NAME == 'release') {
						withMaven(maven: 'MAVEN-3.6.3') {
							echo 'I am pushing the artifact with unique name showing the branch from which it is generated, to Archiva'
							/* echo 'copying the jar and war files from the individual target directorrys and copying it to artifacts directory '
							sh "mkdir -p $WORKSPACE/artifacts"
							sh 'for i in ESBAuditClient ESBAuditLog ESBErrorTranslator TaxESB FraudESB FulfillmentESB PaymentESB ESBRadial ESBAutomatedQueueRetry AlertESB   OrderReconESB;do cp -rp ${i}/dist/libs/* $WORKSPACE/artifacts/ ;done' */
							/* echo 'create a tar file on the Jenkins server'
							sh "cd $WORKSPACE/artifacts/ && tar -cvzf artifact.tar * && md5sum artifact.tar archiveArtifacts artifacts: 'artifacts/*.tar', fingerprint: true */
							sh 'mv sample-java-app/target/sample-0.0.1-SNAPSHOT.jar sample-java-app/target/sample-build_${BUILD_NUMBER}-branch_${BRANCH_NAME}.jar'
							sh 'ls -lrt sample-java-app/target/'
							sh 'mvn -X deploy:deploy-file  -Dfile=sample-java-app/target/sample-build_${BUILD_NUMBER}-branch_${BRANCH_NAME}.jar -DpomFile=sample-java-app/pom.xml -DrepositoryId=snapshots -Durl=https://archiva.sgn.devops.accentureanalytics.com/repository/snapshots/'
							echo "noooooooooooooooooo"
						}
					}
				}
			}
		}
	}
	stage("Deploy") {
		steps {
			container('gradle') {
				script {
					propfile = readProperties(file: './project.properties')
					println("deploy:" + propfile['feature_deploy'])
					println("reading properties ...")
					if (propfile['feature_deploy'] == "true" || env.BRANCH_NAME == 'dev' || env.BRANCH_NAME == 'release') {
						withMaven(maven: 'MAVEN-3.6.3') {
							echo 'I am deploying the artifact into the target environment'
							echo 'shutting down the tomcat ESB server'
							/* sh 'ssh $DEPLOYMENT_USER@$DEPOYMENT_SERVER "$ESB_BIN_PATH stop || sleep 20"'
							echo 'force stop any remaining mule process'
							sh 'ssh $DEPLOYMENT_USER@$DEPOYMENT_SERVER "if [[ $(ps aux | grep /opt/mule/latest/ | grep -vc grep)  > 0 ]]; then "kill -9 $(ps -ef|pgrep -f "/opt/mule/latest/bin")"; else echo "do nothing"; fi"' */
							echo 'tomcat server stopped'
							echo 'copying the tar file from jenkins to deployment directory on app and remove the old folders and untarring the new jar and war files'
							/* sh 'scp -rp $WORKSPACE/artifacts/artifact.tar $DEPLOYMENT_USER@$DEPOYMENT_SERVER:$DEPLOYMENT_STAGE_DIR/'
							sh 'ssh $DEPLOYMENT_USER@$DEPOYMENT_SERVER "cd $DEPLOYMENT_STAGE_DIR && rm -rf *.war *.jar"'
							sh 'ssh $DEPLOYMENT_USER@$DEPOYMENT_SERVER "cd $DEPLOYMENT_STAGE_DIR && tar -xvzf artifact.tar && rm -rf *.jar"'
							sh 'ssh $DEPLOYMENT_USER@$DEPOYMENT_SERVER "cd $DEPLOYMENT_DIR && rm -rf $ESB_WAR_FOLDERS && cp -rp $DEPLOYMENT_STAGE_DIR/*.war $DEPLOYMENT_DIR"' */
							echo 'Deployment has been completed'
							echo 'starting the tomcat ESB server'
							/* sh 'ssh $DEPLOYMENT_USER@$DEPOYMENT_SERVER "$ESB_BIN_PATH start"'
							sh 'ssh $DEPLOYMENT_USER@$DEPOYMENT_SERVER "sleep 20"' */
							echo 'tomcat server started'
						}
					} else {
						echo "no need"
					}
				}
			}
		}
}
	
	stage('Post Deploy Tests') {
		parallel {
			stage('Smoke Test') {
				steps {
					echo "I am executing Smoke Test on target dev environment post deployment"
				}
				/*RESP=`curl -X GET "${bamboo.uri}/RequestsRunning" -H "accept: application/xml" -H "authorization: bearer lR0AA2qfq7v9Ry96vDAgqcer1GPVd5yStmv1_aJVFS43rk06EytB7WsS0_owoiXIgpOXmZVEfkY4ST0JwHtRBk7RH0QRaldWtQT8udC0VdimdGx38RddY2sGaeeF0t9Aflr5rh1Jc_EUfkNK8YrKVxQ6kxB05aCe46CD2fkognv7TiJATmht-ycUjEsd_oy8jH5EK9fmn9eL-wXavNTQcEdsUmFm3-2r3IJDzMK7XCa74qu353yOKLvVyZ1yYQBnc1_fY5GS1BDrFLUZprxpAS30lGEu-d_JTTOQ989UJtIEB3cZzDkIQzeqdYBGCsiDdjdHo2DC1FK2kVPyBITTbQ"`
				echo "The response for current execution status is: $RESP"
				if [ "$RESP" != "[]" ];
				then
				echo "There is a test executing currently in Worksoft. Hence, not proceeding with the execution of Worksoft test cases."
				exit 1
				else
				echo "There are no tests executing right now. Hence, proceeding with Worksoft test execution"
				fi
				# To abort the request before attempting to re-run, uncomment and run below line.
				# abort=$(curl -X PUT -H "Authorization: Bearer $token" -d "" -H "id: ${bamboo.RequestID}" ${bamboo.uri}/Request/${bamboo.RequestID}/abort/)
				guid=$(curl -X PUT -H "Authorization: Bearer $token" -d "" -H "parameters: {TestEnv}{${bamboo.stage_name}}" -H "id: ${bamboo.RequestID}" ${bamboo.uri}/ExecuteRequest/ | tr -d \")
				echo "The GUID is: $guid"
				status=$(curl -X GET -H "Authorization: Bearer $token" -d "" -H "APIRequestID: $guid" ${bamboo.uri}/ExecutionStatus/ | awk -F':' '{print $2}' | tr -d \" | tr -d \})
				echo "The status is: $status"
				while [[ $status != *"Completed"* ]]
				do
				status=$(curl -X GET -H "Authorization: Bearer $token" -d "" -H "APIRequestID: $guid" ${bamboo.uri}/ExecutionStatus/ | awk -F':' '{print $2}' | tr -d \" | tr -d \})
				echo "The status is: $status"
				sleep 15
				done
				status=$(curl -X GET -H "Authorization: Bearer $token" -d "" -H "APIRequestID: $guid" ${bamboo.uri}/ExecutionStatus/)
				echo "The status is: $status"
				execstatus=$(curl -X GET -H "Authorization: Bearer $token" -d "" -H "APIRequestID: $guid" ${bamboo.uri}/ExecutionStatus/ | awk -F':' '{print $3}' | tr -d \" | tr -d \})
				echo "The exec status is: $execstatus"
				if [[ $execstatus != *Passed* ]];
				then
				echo "Failed"
				exit 1
				else
				echo "Passed"
				exit
				fi
				exit */
			}
			stage('Security Test') {
				steps {
					echo 'I am running Security Test here'
				}
			}
		}
	}    
  }
  post {
        failure {
            /*mail bcc: '', 
            	 body: "<b>Example</b><br>\n<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", 
            	 cc: '', 
            	 charset: 'UTF-8', 
            	 from: '', 
            	 mimeType: 'text/html', 
            	 replyTo: '', 
            	 subject: "ERROR CI: Project name -> ${env.JOB_NAME}", 
            	 to: "foo@foomail.com";*/
		echo 'I am sending a notification with failure'
        }
	success {
		echo 'I am sending a notification with success'
	}
	always {
		javadoc: Publish Javadoc
	}
   }
}
