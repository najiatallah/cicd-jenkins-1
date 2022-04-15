pipeline {
	    agent any

	        // Environment Variables
	        environment {
				MAJOR = '1'
				MINOR = '1'
				//Orchestrator Services
				UIPATH_ORCH_URL = "https://cloud.uipath.com/"
				UIPATH_ORCH_LOGICAL_NAME = "ceranic"
				UIPATH_ORCH_TENANT_NAME = "UiPathDefault"
				UIPATH_ORCH_FOLDER_NAME_DEV = "DEV"
				UIPATH_ORCH_FOLDER_NAME_TEST = "TEST"
				UIPATH_ORCH_FOLDER_NAME_PROD = "PROD"
			}
	

	    stages {
	
	        // Printing Basic Information
	        stage('Preparing'){
	            steps {
	                echo "Jenkins Home ${env.JENKINS_HOME}"
	                echo "Jenkins URL ${env.JENKINS_URL}"
	                echo "Jenkins JOB Number ${env.BUILD_NUMBER}"
	                echo "Jenkins JOB Name ${env.JOB_NAME}"
	                echo "GitHub BranchName ${env.BRANCH_NAME}"
	                checkout scm
	            }
	        }
			

			// Building Tests
	        stage('Build Tests') {
	            steps {
	                echo "Building package with ${WORKSPACE}"
	                UiPathPack (
	                      outputPath: "Output\\${JOB_NAME}\\Tests\\${BUILD_NUMBER}",
						  outputType: 'Tests',
	                      projectJsonPath: "project.json",
	                      version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
	                      useOrchestrator: false,
						  traceLevel: 'None'
					)
	            }
	        }

	
			// Deploy Stages
	        //stage('Deploy Tests') {
	        //    steps {
	        //        echo "Deploying ${BRANCH_NAME} to orchestrator"
	        //        UiPathDeploy (
			//			packagePath: "Output\\${JOB_NAME}\\Tests\\${BUILD_NUMBER}",
			//			orchestratorAddress: "${UIPATH_ORCH_URL}",
			//			orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
			//			folderName: "${UIPATH_ORCH_FOLDER_NAME_DEV}",
			//			environments: '',
			//			credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'CeranicApiUserKey'),
			//			traceLevel: 'None',
			//			entryPointPaths: 'Main.xaml'
			//		)
	         //   }
			//}


			// Test Stages
	        stage('Perform Tests') {
	            steps {
	                echo 'Testing the workflow...'
					UiPathTest (
						  // testTarget: [$class: 'TestSetEntry', testSet: "cicd-jenkins-1_Tests"],
						  testTarget: TestProject(environments: '', testProjectPath: "project.json"),
						  orchestratorAddress: "${UIPATH_ORCH_URL}",
						  orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
						  folderName: "${UIPATH_ORCH_FOLDER_NAME_DEV}",
						  timeout: 10000,
						  traceLevel: 'None',
						  testResultsOutputPath: "Output\\${JOB_NAME}\\Tests\\${BUILD_NUMBER}\\result.xml",
						  credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'CeranicApiUserKey'),
						  parametersFilePath: ''
					)
	            }
			}


			// Building Package
	        stage('Build Process') {
				when {
					expression {
						currentBuild.result == null || currentBuild.result == 'SUCCESS' || currentBuild.result != 'UNSTABLE'
						}
				}
				steps {
					echo "Building package with ${WORKSPACE}"
					UiPathPack (
						  outputPath: "Output\\${JOB_NAME}\\Builds\\${BUILD_NUMBER}",
						  projectJsonPath: "project.json",
						  version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
						  useOrchestrator: false,
						  traceLevel: 'None'
					)
				}
	        }			
			
			
	         // Deploy to Production Step
	        stage('Deploy Process') {
				when {
					expression {
						currentBuild.result == null || currentBuild.result == 'SUCCESS' || currentBuild.result != 'UNSTABLE'
						}
				}
				steps {
	                echo 'Deploying process to orchestrator...'
	                UiPathDeploy (
						packagePath: "Output\\${JOB_NAME}\\Builds\\${env.BUILD_NUMBER}",
						orchestratorAddress: "${UIPATH_ORCH_URL}",
						orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
						folderName: "${UIPATH_ORCH_FOLDER_NAME_DEV}",
						environments: '',
						credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'CeranicApiUserKey'),
						traceLevel: 'None',
						entryPointPaths: 'Main.xaml'
					)
				}   
			}	
		
	    }


	    // Options
	    options {
	        // Timeout for pipeline
	        timeout(time:80, unit:'MINUTES')
	        skipDefaultCheckout()
	    }
	

	    // 
	    post {
	        success {
	            echo 'Deployment has been completed!'
	        }
	        failure {
	          echo "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.JOB_DISPLAY_URL})"
	        } 
	    }

	}