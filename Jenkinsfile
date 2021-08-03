def REPOSITORY_NAME = 'psi-asde-batch-6' // Name of the repository in AWS ECR or Docker Hub
def SONAR_SERVER_NAME = 'sonarqube' // SonarQube Server Name defined in Jenkins Configuration
def SONAR_PROJECT_KEY = 'sonarqube' // SonarQube Project Key
def QUALITY_GATE_TIMEOUT = 10 // Quality Gate timeout time in minutes
def RECIPIENT_EMAIL = 'vikasmalinda456@gmail.com' // Email Address of Recipient

pipeline {
  agent any

  options {
    buildDiscarder(logRotator(numToKeepStr: '4', daysToKeepStr: '7', artifactDaysToKeepStr: '7', artifactNumToKeepStr: '4'))
  }

  tools { nodejs 'npm' }

  stages {
    stage('Build') {
      steps {
      //  bat 'echo Y | rmdir /s node_modules'
        bat 'npm install '
      }
    }
    stage(' Test') {
      steps {
        bat 'npm run test --passWithNoTests'
      }
    }
    stage('Code Coverage') {
      steps {
        bat 'npm run coverage'
      }
    }
    stage('SonarQube Analysis') {
      steps {
        script {
          def scannerHome = tool 'SonarScanner 4.0'

          withSonarQubeEnv(SONAR_SERVER_NAME) {
            // If you have configured more than one global server connection,
            //you can specify its name
            bat "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.sources=. -Dsonar.exclusions=node_modules/**"
          }
        }
      }
    }
    stage('Quality Gate') { // SonarQube Quality Gate Stage works only when Webhook enabled
      steps {
        // Timeout will abort pipeline job after specified time
        // Useful when Webhook not recieved
        timeout(time: QUALITY_GATE_TIMEOUT, unit: 'MINUTES') {
          // Runs the groovy script as a step
          script {
            // Waits for Quality Gate Results through Webhook
            qualityGateResult = waitForQualityGate()
            println qualityGateResult.status// Quality Gate Status (Can be used for conditional equations)
            println qualityGateResult.conditions
            if (qualityGateResult.status != 'OK') {
              error 'Quality Gate Failed'
            }
          }
        }
      }
    }
    stage ('Docker Build') {
      steps {
        //sh "docker build . -t ${REPOSITORY_NAME}/${MICROSERVICE_NAME}"
        echo 'Build'
      }
    }
    stage ('Deploy') {
      steps {
        echo 'Deploy Docker Image in AWS '
      }
    }
  }
  post {
    always {
      // Send Notification on any build status (SUCCESS, FAILURE, UNSTABLE)
      sendNotification(RECIPIENT_EMAIL)
    }
  }
}

def sendNotification(email) {
  def buildName = "Job '${JOB_NAME}-${BUILD_NUMBER}'"
  def buildTime = currentBuild.duration / 1000
  def subject = "${currentBuild.currentResult}: ${buildName}"
  def body = "Build Time: ${buildTime}s\nBuild URL: ${BUILD_URL}\nJob URL: ${JOB_URL}"
  def color = (currentBuild.currentResult == 'SUCCESS') ? 'good' : 'danger'

  emailext body: body, to: email, subject: subject
}
