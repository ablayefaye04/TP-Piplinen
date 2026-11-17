
pipeline {
    agent any

    tools {
        // Doit correspondre exactement au nom configuré dans Jenkins
        maven 'Maven3'
    }

    options {
        timeout(time: 2, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Environment') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Build & Tests') {
            steps {
                echo 'Compilation et execution des tests Maven...'
                sh 'mvn -B -ntp clean verify'
            }
            post {
                always {
                    junit testResults: 'target/surefire-reports/*.xml',
                          allowEmptyResults: false

                    archiveArtifacts(
                        artifacts: 'target/site/jacoco/**',
                        allowEmptyArchive: true
                    )
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts(
                    artifacts: 'target/*.jar',
                    fingerprint: true
                )
            }
        }

        stage('Deploy') {
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/main' ||
                           env.BRANCH_NAME == 'main'
                }
            }
            steps {
                sh 'chmod +x deploy.sh && ./deploy.sh'
            }
        }
    }

    post {
        success {
            echo 'Pipeline OK : build, tests et deploiement reussis'
        }

        failure {
            echo 'Pipeline en echec : consultez la Console Output'
        }

        always {
            echo 'Execution du pipeline terminee.'
        }
    }
}
