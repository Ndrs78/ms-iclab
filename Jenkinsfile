def slackMessageCommon = "[Grupo 4][Pipeline CI][Branch: ${env.BRANCH_NAME}][Build: ${env.BUILD_NUMBER}]"

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh('./mvnw clean compile -e')
            }
        }
        stage('Test') {
            steps {
                sh('./mvnw test -e')
            }
        }
        stage('Create package') {
            steps {
                sh('./mvnw package -e')
            }
            post {
                success {
                    archiveArtifacts(artifacts: 'build/*.jar')
                }
            }
        }
        stage('Make a test request') {
            steps {
                sh('''nohup bash mvnw spring-boot:run &
                SPRING_BOOT_PID="$!"
                sleep 5
                curl -X GET "http://localhost:8081/rest/mscovid/test?msg=testing"
                kill $SPRING_BOOT_PID''')
            }
        }
    }
    post {
        success {
            slackSend(channel: 'lab-ceres-mod4-sec2-status', color: 'good', message: "${slackMessageCommon}[Result: SUCCESS]")
        }
        failure {
            slackSend(channel: 'lab-ceres-mod4-sec2-status', color: 'danger', message: "${slackMessageCommon}[Result: FAILED]")
        }
    }
}
