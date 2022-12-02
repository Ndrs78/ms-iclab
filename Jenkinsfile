def lastStage
def slackMessageCommon = "[Grupo 4][Pipeline CI][Branch: ${env.BRANCH_NAME}][Build: ${env.BUILD_NUMBER}]"

pipeline {
    agent any
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
        stage('Create package') {
            steps {
                script { lastStage = env.STAGE_NAME }
                sh('./mvnw package -e')
            }
            post {
                success {
                    archiveArtifacts(artifacts: 'build/*.jar')
                }
            }
        }
        stage("Sonar: Análisis SonarQube"){
            steps {
                withSonarQubeEnv('sonarqube') {
                sh('./mvnw verify sonar:sonar -Dsonar.projectKey=ceres-grupo4')
                }
            }
        }
        stage ('Subir Nexus') {
            steps{
                nexusPublisher nexusInstanceId: 'nexus', 
                nexusRepositoryId: 'Lab4_devops-nexus', 
                packages: [[$class: 'MavenPackage', 
                mavenAssetList: [[classifier: '', extension: '', 
                filePath: '/var/jenkins_home/workspace/Ejercicio4/build/DevOpsUsach2020-0.0.1.jar']], 
                mavenCoordinate: [artifactId: 'ceres_4', 
                groupId: 'Usach_ceres_4', packaging: 'jar', 
                version: '1.1.1']]]
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
                sh(
                    "java -jar \"DevOpsUsach2020-${env.PROJECT_VERSION}.jar\" & " + 
                    '''JAVA_PID="$!"
                    sleep 5
                    curl -X GET "http://localhost:8081/rest/mscovid/test?msg=testing"
                    kill $JAVA_PID'''
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
