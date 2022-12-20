def lastStage
def slackMessageCommon = "[Grupo 4][Pipeline CI][Branch: ${env.BRANCH_NAME}][Build: ${env.BUILD_NUMBER}]"

pipeline {
  agent any
  environment {
    // https://stackoverflow.com/a/60023743
    PROJECT_VERSION = """${ sh(script:'./mvnw org.apache.maven.plugins:maven-help-plugin:3.2.0:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true) }"""
  }
  stages {
    stage('Build') {
      steps {
        script { lastStage = env.STAGE_NAME }
        sh('./mvnw clean compile -e')
      }
    }
    stage('Test') {
      steps {
        script { lastStage = env.STAGE_NAME }
        sh('./mvnw test -e')
      }
    }
    stage("Run SonarQube on code"){
      steps {
        withSonarQubeEnv('sonarqube') {
        sh('./mvnw verify sonar:sonar -Dsonar.projectKey=ceres-grupo4')
        }
      }
    }
    stage('Create package') {
      steps {
        script { lastStage = env.STAGE_NAME }
        sh('./mvnw package -e')
      }
    }
    stage ('Upload to Nexus') {
      steps{
        nexusPublisher(
          nexusInstanceId: 'ceres-nexus', 
          nexusRepositoryId: 'maven-ceres-grupo4', 
          packages:[[
            $class: 'MavenPackage', 
            mavenAssetList: [[
              filePath: "build/DevOpsUsach2020-${env.PROJECT_VERSION}.jar"
            ]], 
            mavenCoordinate: [
              groupId: 'cl.usach',
              artifactId: 'DevOpsUsach2020',
              packaging: 'jar',
              version: "${env.PROJECT_VERSION}"
            ]
          ]]
        )
      }
    }
    stage('Pull package and make test request') {
      steps {
        script { lastStage = env.STAGE_NAME }
        withCredentials([usernameColonPassword(credentialsId: 'nexus-user', variable: 'nexusLogin')]) {
          sh(
            'curl -X GET -u "$nexusLogin" -O ' + 
            "\"http://nexus:8081/repository/maven-ceres-grupo4/cl/usach/DevOpsUsach2020/${env.PROJECT_VERSION}/DevOpsUsach2020-${env.PROJECT_VERSION}.jar\""
          )
        }
        sh("nohup java -jar \"DevOpsUsach2020-${env.PROJECT_VERSION}.jar\" & ")
        sh("sleep 5")
        sh('curl -X GET "http://localhost:8081/rest/mscovid/test?msg=testing"')
        // No need to kill the process. Jenkins's ProcessTreeKiller will do it for us.
      }
    }
  
    stage("Testear Artefacto - Newman "){
            steps {
                script{
                    sh "newman run ejemplo-maven.postman_collection.json -n 5  --delay-request 1000"
                }
            }
        }
  post {
    success {
      slackSend(
        channel: 'lab-ceres-mod4-sec2-status',
        color: 'good',
        message: "${slackMessageCommon}[Result: SUCCESS] (<${env.BUILD_URL}|Open>)")
    }
    failure {
      slackSend(
        channel: 'lab-ceres-mod4-sec2-status',
        color: 'danger',
        message: "${slackMessageCommon}[Stage: ${lastStage}][Result: FAILED] (<${env.BUILD_URL}|Open>)")
    }
  }
 }
}
