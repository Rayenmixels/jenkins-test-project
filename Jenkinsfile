pipeline {
    agent any

    stages {
        stage('GIT') {
            steps {
                git branch: 'main',
                    changelog: false,
                    credentialsId: 'jenkins-github',
                    url: 'https://github.com/Rayenmixels/jenkins-test-project.git'
            }
        }

        stage('MAVEN Build') {
            steps {
                sh 'mvn -B clean verify'
            }
        }

        stage('SONARQUBE') {
            environment {
                SONAR_HOST_URL = 'http://192.168.50.4:9000'
                SONAR_AUTH_TOKEN = credentials('sonarqube')
            }
            steps {
                sh 'mvn sonar:sonar -Dsonar.projectKey=devops_git -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONAR_AUTH_TOKEN'
            }
        }

        stage('OWASP Dependency-Check Vulnerabilities') {
            steps {
                dependencyCheck additionalArguments: '''
                    -o '.'
                    -s '.'
                    -f 'ALL'
                    --prettyPrint
                ''', odcInstallation: 'OWASP Dependency-Check Vulnerabilities'

                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }

        stage('Secrets Scan with Gitleaks') {
            steps {
                script {
                    sh '''
                        docker run --rm -v ${WORKSPACE}:/repo zricethezav/gitleaks:latest \
                        detect --source /repo --report-format json --report-path /repo/gitleaks-report.json
                    '''
                }

                archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            slackSend(
                channel: '#nouveau-canal',
                color: (currentBuild.currentResult == 'SUCCESS') ? 'good' : 'danger',
                message: "*${currentBuild.currentResult}*: Job ${env.JOB_NAME} (Build #${env.BUILD_NUMBER})\n${env.BUILD_URL}",
                tokenCredentialId: 'slack-token'
            )
        }
    }
}
