pipeline {

  agent {
       node {
           label 'mavenslaves'
       }
   }

   options { disableConcurrentBuilds() }


  stages {

    stage('Checkout') {
        steps {
            script{
                deleteDir()
                checkout scm
                echo "Sync Branch ${env.BRANCH_NAME}"
                sh "git checkout ${env.BRANCH_NAME}"

            }
        }
   }

    stage('Test') {
      steps {
        echo '### Test ###'
	sh "coverage run --source testing -m unittest discover && coverage report"
      }
    }

   stage('SonarQube analysis') {
       steps {
           script {
               env.scannerHome = tool'SonarQubeScanner'
               withSonarQubeEnv('SonarQubeScanner') {
                   sh "${env.scannerHome}/bin/sonar-scanner " + "-Dproject.settings=.sonarqube.properties"
               }
           }
       }
   }

   stage('Quality Gate') {
       steps {
           script {
               timeout(time: 15, unit: 'MINUTES') {
                   def qg = waitForQualityGate()
                   if ( qg.status == 'WARN') {
                       slackSend botUser: true, baseUrl: "${env.SlackBaseUrl}", channel: "${env.SlackChannel}", color: '#bb2124', message: ":warning: Pipeline passed Quality Gate but with warnings! \nCheck Quality Gate result <http://sonarqube-devtools-production-828124840.us-east-1.elb.amazonaws.com/dashboard?id=app%3Afoxsportsla|here>", token: "${env.SlackToken}"
                       //error "Pipeline aborted due to quality gate failure: ${qg.status}"
                   }
                   if ( qg.status == 'ERROR') {
                       slackSend botUser: true, baseUrl: "${env.SlackBaseUrl}", channel: "${env.SlackChannel}", color: '#bb2124', message: ":x: Pipeline aborted due to quality gate failure: ${qg.status} \nCheck Quality Gate result <http://sonarqube-devtools-production-828124840.us-east-1.elb.amazonaws.com/dashboard?id=app%3Afoxsportsla|here>", token: "${env.SlackToken}"
                       error "Pipeline aborted due to quality gate failure: ${qg.status}"
                   }
               }
           }
       }
   }
  }
}
