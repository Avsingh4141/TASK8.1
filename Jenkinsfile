pipeline {
    agent any

    tools { 
        nodejs "NodeJS_24.9.0" // name from Global Tool Configuration
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Avsingh4141/8.2CDevSecOps.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install --no-audit --no-fund || exit /b 0'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm test > test-output.txt 2>&1 || exit /b 0'
                archiveArtifacts artifacts: 'test-output.txt', allowEmptyArchive: true
            }
        }

        stage('NPM Audit (Security Scan)') {
            steps {
                bat 'npm audit --json > npm-audit.json || exit /b 0'
                archiveArtifacts artifacts: 'npm-audit.json', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            script {
                // Zip logs if they exist
                if (fileExists('test-output.txt') || fileExists('npm-audit.json')) {
                    powershell """
                    if (Test-Path pipeline-logs.zip) { Remove-Item pipeline-logs.zip }
                    Compress-Archive -Path test-output.txt, npm-audit.json -DestinationPath pipeline-logs.zip -Force
                    """
                }
                
                // Send email
                if (fileExists('pipeline-logs.zip')) {
                    emailext(
                        to: 'jk8237405@gmail.com',
                        subject: "Jenkins Pipeline: ${currentBuild.fullDisplayName} - ${currentBuild.currentResult}",
                        body: """Build Status: ${currentBuild.currentResult}
Project: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Check Build: ${env.BUILD_URL}

Attached are the logs and audit results.""",
                        attachmentsPattern: 'pipeline-logs.zip',
                        attachLog: true
                    )
                } else {
                    emailext(
                        to: 'jk8237405@gmail.com',
                        subject: "Jenkins Pipeline: ${currentBuild.fullDisplayName} - ${currentBuild.currentResult}",
                        body: """Build Status: ${currentBuild.currentResult}
Project: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Check Build: ${env.BUILD_URL}

No logs to attach.""",
                        attachLog: true
                    )
                }
            }
        }
    }
}
