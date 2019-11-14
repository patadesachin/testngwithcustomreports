pipeline {
    agent any
	tools { 
        maven 'maven3.6' 
        jdk 'jdk8'
    }
	parameters {
		choice(name:'Report_Selection',
			choices:'TestNG_withCentralizedReport\nExtent_Report\nTestNGReport',
			description:'which reports need to use?')
		choice(name:'TestType_Selection',
			choices:'Smoke\nRegression',
			description:'which test execution smoke or regression?')
			}

    stages {
		stage('Smoke Test with Centralized Report') {
            steps{
                echo "Pulling value from ${params.Report_Selection}"
                echo "Pulling value from ${params.TestType_Selection}"

                sh 'mvn clean test -P"echo ${params.Report_Selection}" -Dgroups="echo ${params.TestType_Selection}"'                         }
            }
        }

		stage('Functional Tests with Centralized Report') {
            steps{
                echo ''
                sh 'mvn clean test -P${params.Report_Selection} -Dgroups=${params.TestType_Selection}'
            }
        }
        
        stage("sonar_static_check"){
            steps{
		withSonarQubeEnv('MySonarQube') {
                    // Optionally use a Maven environment you've configured already
                    sh 'mvn -f clean sonar:sonar -Dmaven.test.skip=true'
                }
            }

        }
	
	stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

		
        stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "ARTIFACTORY_SERVER",
                    url: "http://artifactory:8081/artifactory",
		    credentialsId: 'admin.jfrog'
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )
            }
        }

        stage ('Build & Upload Artifact') {
            steps {
                rtMavenRun (
                    tool: "maven3.6", // Tool name from Jenkins configuration
                    pom: 'pom.xml',
                    goals: 'clean install -Dmaven.test.skip=true',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
        }

        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "ARTIFACTORY_SERVER"
                )
            }
        }
    }
}
