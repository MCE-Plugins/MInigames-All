pipeline {
   agent any
   tools {
      maven 'maven3.5.0'
      jdk 'jdk8'
   }
   stages {
      stage ('Initialization') {
          steps {
              echo 'Preparing for build'
          }
      }
      
      stage ('Build') {
          steps {
              sh 'mvn clean install' 
              archiveArtifacts artifacts: '**/target/*.zip', fingerprint: true
              archiveArtifacts artifacts: '**/target/*.bz2', fingerprint: true
              archiveArtifacts artifacts: '**/target/*.gz', fingerprint: true
          }
          post {
              failure {
                  mail to: 'mep_eisen@web.de', subject: 'build of Minigames-All failed', body: 'build of Minigames-All failed'
              }
          }
      }
      
      stage ('Upload') {
        steps {
        	script {
        		env.BUILDTYPE = readPomVersion().endsWith("-SNAPSHOT") ? "snapshots" : "releases";
        	}
            sh "/srv/hudson/upload_minigames_all.sh ${env.BUILDTYPE} target 1"
      	}
      }
      
      stage ('Deploy') {
          when {
              expression {
                  currentBuild.result == null || currentBuild.result == 'SUCCESS'
              }
          }
          steps {
              sh 'mvn -Dmce.deployment=true -Dmaven.test.skip=true -DskipTests deploy'
          }
          post {
              failure {
                  mail to: 'mep_eisen@web.de', subject: 'deploy of Minigames-All failed', body: 'deploy of Minigames-All failed'
              }
          }
      }
   }
}

def readPomVersion() {
	// Reference: http://stackoverflow.com/a/26514030/1851299
	sh "mvn --quiet --non-recursive -Dexec.executable='echo' -Dexec.args='\${project.version}' org.codehaus.mojo:exec-maven-plugin:1.3.1:exec > pom.project.version.txt"
	pomProjectVersion = readFile('pom.project.version.txt').trim()
	sh "rm -f pom.project.version.txt"
	echo "Current POM version: ${pomProjectVersion}"
	return pomProjectVersion
}
