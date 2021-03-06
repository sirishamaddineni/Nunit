pipeline {
    agent any

    stages 
	{
		stage( 'Checkout Source' ) 
		{
		//Checkout source code from GIT
		  steps
		  {
			// workaround to get GIT plugin environment Variables, we need to collect the checkout command output, which is a map that contains them
			// https://issues.jenkins-ci.org/browse/JENKINS-35230
			  script
			  {
				scm_map = checkout scm
				GIT_BRANCH = scm_map['GIT_BRANCH']
				// get just the branch name minus the remote only splitting on first
				// match in case the rest of the branch has more '/' chars.
				GIT_BRANCH_NAME = GIT_BRANCH.split('/',2)[1]				
			  }				
				 bat "echo 'Checkout of GIT branch: ${GIT_BRANCH}'"
			     bat "echo 'GIT_BRANCH_NAME: ${GIT_BRANCH_NAME}'"
			}
		}//End Checkout Source   
		
		stage( 'Build' ) 
		{
		//Build source code
		  steps
		  {
			bat "\"${tool 'MSBuild 14'}\" NunitDemo.sln /p:Configuration=Release /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"   
			}
		}//End Build source code 
		
		stage( 'Test' ) 
		{
		//Build source code
		  steps
		  {
			bat '''"C:\\Program Files (x86)\\Jenkins\\workspace\\Poc\\packages\\OpenCover.4.6.519\\tools\\OpenCover.Console.exe" -target:"C:\\Program Files (x86)\\Jenkins\\workspace\\Poc\\packages\\NUnit.ConsoleRunner.3.7.0\\tools\\nunit3-console.exe" -targetargs:"/work:Reporting --out:TestResult.txt .\\NunitDemo.Test\\bin\\Release\\NunitDemo.Test.dll" -register:administrator -filter:"+[*]* -[NunitDemo.Test]*" -output:"CodeCoverageResult.xml"
				"C:\\Program Files (x86)\\Jenkins\\workspace\\Poc\\packages\\ReportGenerator.3.1.0\\tools\\ReportGenerator.exe" "-reports:CodeCoverageResult.xml" "-targetdir:CodeCoverageReport"
				"C:\\Program Files (x86)\\Jenkins\\workspace\\Poc\\packages\\ReportUnit.1.2.1\\tools\\ReportUnit.exe" "Reporting" "Reporting/Result" "'''
	                              
		  }
			
		}
		//End Build source code 
		stage ('publish') {
		steps{
		        
				publishHTML([allowMissing: false,
					     alwaysLinkToLastBuild: true,
					     keepAll: false,
					     reportDir: 'Reporting/Result',
					     reportFiles: 'index.html',
					     reportName: 'NUnit Test Report',
					     reportTitles: 'NUnit Test Report'])
		}
		}
							
		stage( 'Report' ) {
		       
			steps
			{
				publishHTML([allowMissing: false,
					     alwaysLinkToLastBuild: true,
					     keepAll: false,
					     reportDir: 'CodeCoverageReport',
					     reportFiles: 'index.htm',
					     reportName: 'Code Coverage Report',
					     reportTitles: 'Code Coverage Report'])
			}
		}
						     
		
		stage( 'Package into zip file' ) 
		{
		//Build source code
		  steps
		  {
			bat '"C:\\Program Files\\7-Zip\\7z.exe" a  -r "DemoNunit.zip" -w NunitDemo.Test\\bin\\Release\\* -mem=AES256'
			}
		}//End Build source code
		
		stage( "IQ Scans") {
		  steps{
			bat "echo 'Uploading to IQ: 'DemoNunit' stage:'release' file: 'DemoNunit.zip'"
			nexusPolicyEvaluation failBuildOnNetworkError: false,
				iqApplication: 'IQ_App',
				iqScanPatterns: [[scanPattern: 'DemoNunit.zip' ]],
				iqStage: 'release',
				jobCredentialsId: ''
		  }
		} // stage	
		stage( "Upload to Nexus" )
		{
		steps{
		nexusArtifactUploader artifacts: [[artifactId: 'DemoNunit', classifier: '', file: 'DemoNunit.zip', type: 'zip']],
		credentialsId: '5aed14d0-99ce-4bd4-bbad-b6e3014c4e28', 
		groupId: 'maven-public', 
		nexusUrl: 'localhost:9091', 
		nexusVersion: 'nexus3', 
		protocol: 'http', 
		repository: 'Repo1', 
		version: '1.0'
		}
	     }
		stage( "email" )
		{
			steps{
	                                mail bcc: '',
					body: 'notification',
					cc: '', 
					from: 'sirishamaddineni25@gmail.com', 
					replyTo: 'suresh.suri976@gmail.com', 
					subject: 'test', 
					to: 'sureshyaramasi@gmail.com'
			}
		}

	}
	}
