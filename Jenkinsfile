def getRepoURL() {
  sh "git config --get remote.origin.url > .git/remote-url"
  return readFile(".git/remote-url").trim()
}

def getCommitSha() {
  sh "git show-ref -s $GIT_BRANCH > .git/current-commit"
  return readFile(".git/current-commit").trim()
}

def updateGithubCommitStatus() {
  repoUrl = getRepoURL()
  commitSha = getCommitSha()

  step([
    $class: 'GitHubCommitStatusSetter',
    reposSource: [$class: "ManuallyEnteredRepositorySource", url: repoUrl],
    commitShaSource: [$class: "ManuallyEnteredShaSource", sha: commitSha],
    errorHandlers: [[$class: 'ShallowAnyErrorHandler']],
    statusResultSource: [
      $class: 'ConditionalStatusResultSource',
      results: [
        [$class: 'BetterThanOrEqualBuildResult', result: 'SUCCESS', state: 'SUCCESS'],
        [$class: 'BetterThanOrEqualBuildResult', result: 'FAILURE', state: 'FAILURE'],
        [$class: 'AnyBuildResult', state: 'FAILURE', message: 'Loophole']
      ]
    ]
  ])
}

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
        always {
            updateGithubCommitStatus()
        }
    }
}
