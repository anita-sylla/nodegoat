pipeline {
    agent any

    environment {
        REPORT_EMAIL = 'antas2076@gmail.com'
    }

    stages {

        stage('Clone du code') {
            steps {
                checkout scm
            }
        }

        stage('Installation des dependances') {
            steps {
                bat 'npm install || exit 0'
            }
        }

        stage('SAST - Analyse Semgrep') {
            steps {
                bat '''
                    if not exist semgrep-reports mkdir semgrep-reports

                    "C:\\Users\\TOSHIBA\\AppData\\Local\\Programs\\Python\\Python313\\Scripts\\semgrep.exe" --config p/javascript --config p/nodejs --config p/owasp-top-ten --config p/secrets --json --output semgrep-reports\\semgrep-report.json --exclude node_modules --exclude test . || exit 0

                    "C:\\Users\\TOSHIBA\\AppData\\Local\\Programs\\Python\\Python313\\Scripts\\semgrep.exe" --config p/javascript --config p/nodejs --config p/owasp-top-ten --config p/secrets --output semgrep-reports\\semgrep-report.txt --exclude node_modules --exclude test . || exit 0

                    type semgrep-reports\\semgrep-report.txt || exit 0
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'semgrep-reports/**',
                             allowEmptyArchive: true

            emailext(
                to: "${REPORT_EMAIL}",
                subject: "Semgrep SAST | ${currentBuild.result} | ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                <html><body style="font-family: Arial;">
                <h2 style="color:#c0392b;">Rapport SAST - Semgrep</h2>
                <table border="1" cellpadding="10" style="border-collapse:collapse;">
                  <tr><td><b>Projet</b></td><td>OWASP NodeGoat</td></tr>
                  <tr><td><b>Build</b></td><td>#${env.BUILD_NUMBER}</td></tr>
                  <tr><td><b>Statut</b></td><td>${currentBuild.result}</td></tr>
                  <tr><td><b>Duree</b></td><td>${currentBuild.durationString}</td></tr>
                  <tr><td><b>Regles</b></td><td>p/javascript, p/nodejs, p/owasp-top-ten, p/secrets</td></tr>
                </table>
                <br>
                <p>Rapport semgrep-report.txt en piece jointe</p>
                <a href="${env.BUILD_URL}">Voir le Build Jenkins</a>
                </body></html>
                """,
                mimeType: 'text/html',
                attachmentsPattern: 'semgrep-reports/semgrep-report.txt'
            )
        }
        success { echo 'Pipeline termine - email envoye' }
        failure { echo 'Pipeline echoue - verifier les logs' }
    }
}