pipeline {
  agent any
  stages {
   
    stage('Initialize') {
      steps {
        timeout(time: 3, unit: 'MINUTES') {
          echo 'waiting 2 seconds ....'
          sleep(unit: 'SECONDS', time: 2)
          git(poll: true, url: 'https://github.com/yaichZied/gameoflife.git', branch: '${BRANCH_NAME}', changelog: true)
   
        }
        
        
      }
    }
    stage('Build') {
      steps {
        script {
          emailext (
            subject: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
            body: """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
            <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
            recipientProviders: [[$class: 'DevelopersRecipientProvider']]
          )
          slackSend (message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})" ,color: '#FFFF00')
          configFileProvider([configFile( fileId: 'c775a584-3f02-4ba0-bfb1-f559bc87178d', variable: 'MAVEN_SETTINGS')]) {
            
            echo "MAVEN_SETTINGS = $MAVEN_SETTINGS"
            
            sh "mvn    clean -U"
            sh "mvn  install -U  "
            sh "mvn  -s $MAVEN_SETTINGS  deploy -U"
   
            sh """
            git config --global user.email zied.yaich5@gmail.com
            git config --global user.name yaichZied
            """
            sh "git clean -df && git reset --hard"
            sh "mvn -s $MAVEN_SETTINGS release:clean release:prepare release:perform -U "
               script {
          sh "mvn dependency:get -X -DremoteRepositories=http://admin:admin123@127.0.0.1:8081/repository/maven-releases -Dartifact=com.wakaleo.gameoflife:gameoflife-web:LATEST:war -Dtransitive=false"
        }
   
          }
        }
        
      }
    }
    stage('SonarQube') {
      steps {
        script {
          configFileProvider([configFile( fileId: 'c775a584-3f02-4ba0-bfb1-f559bc87178d', variable: 'MAVEN_SETTINGS')]) {
            sh" mvn -s $MAVEN_SETTINGS sonar:sonar "
          }
        }
        
      }
    }
    stage('Report') {
      steps {
        junit(testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true, healthScaleFactor: 1)
        archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        timeout(time: 2, unit: 'HOURS') {
          input 'Does this build seems OK ?'
        }
        
      }
    }
    stage('Deployement') {
      steps {
        echo 'depoying app'
      }
    }
  }
  tools {
    maven 'Maven 3.3.9'
    jdk 'jdk8'
  }
  environment {
    VERSION = readMavenPom().getVersion()
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    timeout(time: 60, unit: 'MINUTES')
  }
}
