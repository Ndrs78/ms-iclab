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
}
