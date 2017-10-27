#!groovy​
pipeline {
    environment {
        sqScannerMsBuildHome = tool 'Scanner for MSBuild'
		strProjectName = 'MHR_MVC'
    }
    
    agent any
    
    stages {
        stage('Build + SonarQube analysis') {
            steps {
				bat 'C:\\Services\\nuget.exe restore %strProjectName%.sln'
                withSonarQubeEnv('Huckshome SonarQube Server') {
                    bat '"%sqScannerMsBuildHome%\\SonarQube.Scanner.MSBuild.exe" begin /k:%strProjectName% /n:%strProjectName% /v:1.0.0.%BUILD_NUMBER% /d:sonar.host.url=%SONAR_HOST_URL% /d:sonar.login=%SONAR_AUTH_TOKEN%'
                    bat '"C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Professional\\MSBuild\\15.0\\Bin\\MSBuild.exe" %strProjectName%.sln /t:Build /p:Configuration=Debug'
                    bat '"%sqScannerMsBuildHome%\\SonarQube.Scanner.MSBuild.exe" end'
                }
            }
        }
        
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
					script {
						def qg = waitForQualityGate()
						if (qg.status != 'OK') {
							error "Pipeline aborted due to quality gate failure: ${qg.status}"
						}
					}
                }
            }
        }
        
        stage('Test') {
            steps {
                echo 'No tests'
            }
        }
        
        stage('Build for Release') {
            steps {
				bat '"C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Professional\\MSBuild\\15.0\\Bin\\MSBuild.exe" "%strProjectName%/%strProjectName%.csproj" /T:Rebuild /p:Configuration=Release /p:OutputPath="obj\\Release\\bin" /p:WebProjectOutputDir="obj\\Release" /p:MvcBuildViews=true'
            }
        }
        
        stage('Deploy') {
            steps {
                bat 'del Z:\\Websites\\%strProjectName%\\**'
                bat 'xcopy "%strProjectName%\\obj\\Release\\netcoreapp2.0\\**" "Z:\\Websites\\%strProjectName%\\" /s /y'
            }
        }
    }
}