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
                steps {
                    script {
                            //pipeline-utility-steps
                        dir("build_java"){
                            def   pom = readMavenPom file: "pom.xml";
                                
                            dir("target"){
                                nexusArtifactUploader(
                                    nexusVersion: 'nexus3',
                                    protocol: 'http',
                                    nexusUrl: 'localhost:8081',
                                    groupId: group-ceres-4,
                                    version: '0.0.1',
                                    repository: 'Lab_devops-nexus',
                                    credentialsId: 'useradminnexus',
                                    artifacts: [
                                        [artifactId: 'Lab4_devops',
                                        classifier: '',
                                        filePath: 'build/DevOpsUsach2020-0.0.1.jar',
                                        type: pom.packaging]
                                    ]
                                )
                            }
                        }
                    }
                }
        }    
        stage('Make a test request') {
            steps {
                script { lastStage = env.STAGE_NAME }
                sh('''nohup bash ./mvnw spring-boot:run &
                SPRING_BOOT_PID="$!"
                sleep 5
                curl -X GET "http://localhost:8081/rest/mscovid/test?msg=testing"
                kill $SPRING_BOOT_PID''')
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
