pipeline {
    agent any

    // Use the NodeJS installation you configured in Jenkins Tools
    tools { nodejs "NodeJS 24.9.0" } 

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Avsingh4141/8.2CDevSecOps.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install --no-audit --no-fund'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm test > test-output.txt 2>&1'
                archiveArtifacts artifacts: 'test-output.txt', allowEmptyArchive: true
            }
        }

        stage('NPM Audit (Security Scan)') {
            steps {
                bat 'npm audit --json > npm-audit.json'
                archiveArtifacts artifacts: 'npm-audit.json', allowEmptyArchive: true
            }
        }
    }

    post {
        success {
            powershell """
            if (Test-Path pipeline-logs.zip) { Remove-Item pipeline-logs.zip }
            Compress-Archive -Path test-output.txt, npm-audit.json -DestinationPath pipeline-logs.zip
            """
            archiveArtifacts artifacts: 'pipeline-logs.zip', allowEmptyArchive: true

            emailext (
                to: 'jk8237405@gmail.com',
                subject: "Jenkins Pipeline SUCCESS: ${currentBuild.fullDisplayName}",
                body: """Build Status: SUCCESS
Project: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Check Build: ${env.BUILD_URL}

Attached are the logs and audit results.""",
                attachmentsPattern: 'pipeline-logs.zip',
                attachLog: true
            )
        }

        failure {
            powershell """
            if (Test-Path pipeline-logs.zip) { Remove-Item pipeline-logs.zip }
            Compress-Archive -Path test-output.txt, npm-audit.json -DestinationPath pipeline-logs.zip
            """
            archiveArtifacts artifacts: 'pipeline-logs.zip', allowEmptyArchive: true

            emailext (
                to: 'jk8237405@gmail.com',
                subject: "Jenkins Pipeline FAILURE: ${currentBuild.fullDisplayName}",
                body: """Build Status: FAILURE
Project: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Check Build: ${env.BUILD_URL}

Attached are the logs and audit results.""",
                attachmentsPattern: 'pipeline-logs.zip',
                attachLog: true
            )
        }
    }
}
