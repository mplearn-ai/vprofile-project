def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]
pipeline {
    
	agent any

	tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'Unix11!'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vprofile-maven-central'
        NEXUSIP = '172.31.21.33'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn -s settings.xml clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

	stage('UNIT TEST'){
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

	// stage('INTEGRATION TEST'){
    //         steps {
    //             sh 'mvn verify -DskipUnitTests'
    //         }
    //     }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
            // post {
            //     success {
            //         echo 'Generated Analysis Result'
            //     }
            // }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool "${SONARSCANNER}"
          }

          steps {
            withSonarQubeEnv("${SONARSERVER}") {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
          }
        }

        stage('UPLOAD ARTIFACT') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [
                        [artifactId: 'vproapp',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war']
                    ]
                );
            }
        }

    //     stage("Publish to Nexus Repository Manager") {
    //         steps {
    //             script {
    //                 pom = readMavenPom file: "pom.xml";
    //                 filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
    //                 echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
    //                 artifactPath = filesByGlob[0].path;
    //                 artifactExists = fileExists artifactPath;
    //                 if(artifactExists) {
    //                     echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
    //                     nexusArtifactUploader(
    //                         nexusVersion: NEXUS_VERSION,
    //                         protocol: NEXUS_PROTOCOL,
    //                         nexusUrl: NEXUS_URL,
    //                         groupId: NEXUS_REPOGRP_ID,
    //                         version: ARTVERSION,
    //                         repository: NEXUS_REPOSITORY,
    //                         credentialsId: NEXUS_CREDENTIAL_ID,
    //                         artifacts: [
    //                             [artifactId: pom.artifactId,
    //                             classifier: '',
    //                             file: artifactPath,
    //                             type: pom.packaging],
    //                             [artifactId: pom.artifactId,
    //                             classifier: '',
    //                             file: "pom.xml",
    //                             type: "pom"]
    //                         ]
    //                     );
    //                 } 
	// 	    else {
    //                     error "*** File: ${artifactPath}, could not be found";
    //                 }
    //             }
    //         }
    //     }


    }


    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }

}
