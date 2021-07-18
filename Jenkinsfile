pipeline {
  agent {
    node{
      label 'master'
      customWorkspace "workspace/${env.JOB_NAME}"
    }
  }
  triggers {
    //Croon job time setting
    // pollSCM 'H 15 * * *'
    // To run the build using the github triggers
    issueCommentTrigger(triggerExpression)
  }
  environment {
    //GITHUB_TOKEN = credentials('credential id')
		//JFROG_URL = 'local.jfrog.com/artifactory'
		pom = readMavenPom file: "pom.xml"
		repoPath =  "${pom.groupId.replace("." , "/")}"	    
		//artifactUrl = "https://${JFROG_URL}/reponame/${repoPath}/${pom.artifactId}/${pom.version}/${pom.artifactId}-${pom.version}.jar"
		//ARTIFACT_URL= '$artifactUrl'
		//APP_NAME="${pom.artifactId}-${pom.version}"
		//url = "https://local.docker.repo.name:port"
    //registryCredential = 'dockerhub-credentials'
    //git organisation repo handiling purpose we are using beolw variable
    //repo_name = reponame()
  }
  options {
    buildDiscarder(logRotator(artifactDaysToKeepStr: '30', artifactNumToKeepStr: '5', daysToKeepStr: '30', numToKeepStr: '5'))
  }
  stages {
    stage('Jira-ID Check') {
      agent{
        /* docker {
          image 'docker-repo-url:port/maven:jdk-11.0.7' //container will start from this image
		      args '-v $WORKSPACE:/opt/ -u root'
		      label 'runner-02'
        } */
        label 'master'
      }
      steps {
        script {
          /*if (env.CHANGE_ID) {
            def prTitle = pullRequest.title
            def pattern = ~"CIQES-[0-9]{5,7}"
            println prTitle
            if(prTitle =~ /CIQES-[0-9]{5,7}/) {
              status = "success"
              description = "Title Check passed"
            } else {
              status = "error"
              description = "Not a valid Jira ID"
              autoCancelled = true
              error('Aborting the build')
            }
            pullRequest.createStatus(status: status,
                                     context: 'Jira Title Check',
                                     description: description,
                                     targetUrl: env.JOB_URL) */
          echo "To check the Jira story ID"
          }
        }
      }
    }
    stage('Build') {
      agent{
         /*docker {
          image 'docker-repo-url:port/maven:jdk-11.0.7' //container will start from this image
		      args '-v $WORKSPACE:/opt/ -u root'
		      label 'runner-02'
        } */
        label 'master'
      }
      steps {
        script {
          sh "mvn -f pom.xml deploy -U -DbuildNumber=${BUILD_NUMBER} -DskipTests=false"
          archiveArtifacts artifacts: 'target/*.jar, *.yml'
          stash includes: 'target/**, *.yml', name: 'artifacts'
          stash includes: 'target/surefire-reports, target/site/jacoco/jacoco.xml', name: 'jacoco-artifacts'
          stash includes: '*', name: 'sourceCode'
        }
      }
    }
    stage('Unit Test') { 
			agent{
        /* docker {
          image 'docker-repo-url:port/maven:jdk-11.0.7' //container will start from this image
		      args '-v $WORKSPACE:/opt/ -u root'
		      label 'runner-02'
        } */
        label 'master'
      }
      steps {
        script {
          sh 'mvn -f pom.xml test -DskipTests=false'
        }
      }
    }
    stage('Sonar-Analysis'){
      agent{
        /*docker {
          image 'docker-repo-url:port/maven:jdk-11.0.7' //container will start from this image
		      args '-v $WORKSPACE:/opt/ -u root'
		      label 'runner-02'
        } */
        label 'master'
      }
      environment {
        branch = branchname()
      }
      steps{
        script {
          echo "To run the sonar scan"
         /* unstash 'artifacts'
          def sonarscanner = libraryResource 'sonarscanner.sh'
          withSonarQubeEnv('Sonar') {
            sh sonarscanner
          } */
        }
      }
    }
    stage('Jacoco Result'){
      agent{
        /*docker {
          image 'docker-repo-url:port/maven:jdk-11.0.7' //container will start from this image
		      args '-v $WORKSPACE:/opt/ -u root'
		      label 'runner-02'
        }*/
        label 'master'
      }
      steps{
        script {
          echo "Run the Jacoco result"
          //unstash 'artifacts'
          //jacoco deltaBranchCoverage: '70', deltaClassCoverage: '100', deltaComplexityCoverage: '70', deltaInstructionCoverage: '70', deltaLineCoverage: '70', deltaMethodCoverage: '70', maximumBranchCoverage: '70', maximumClassCoverage: '100', maximumComplexityCoverage: '70', maximumInstructionCoverage: '70', maximumLineCoverage: '70', maximumMethodCoverage: '70'
        }
      }
    }
    /*stage('Artifactory Upload') {
      when{
        beforeAgent true
				branch 'release-*'
      }
      agent{
        docker {
          image 'docker-repo-url:port/maven:jdk-11.0.7' //container will start from this image
		      args '-v $WORKSPACE:/opt/ -u root'
		      label 'maven'
        }
      }
      steps {
        script {
          unstash 'artifacts'
          echo 'Pushing Artifact'
          def server = Artifactory.server 'new-artifactory'
          def uploadSpec = """{
          "files": [{
          "pattern": "target/*.jar",
          "target": "maven-repo-name"
          }]
          }"""
          server.upload(uploadSpec)
        }
		}
		} */
    }
  post {
    always {
      cleanWs deleteDirs: true
      dir("${env.WORKSPACE}@tmp") {
        deleteDir()
      }
      emailext body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
        recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
        to: "jayasankar.boddu@gmail.com,${env.BUILD_USER_EMAIL}",
        subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
    }
  }
}
