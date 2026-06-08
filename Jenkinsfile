pipeline {
    agent any

    environment {
        REPORT_EMAIL  = 'antas2076@gmail.com'
        REPORT_DIR    = 'semgrep-reports'
        SEMGREP_RULES = 'p/javascript p/nodejs p/owasp-top-ten p/secrets'
    }

    stages {

        stage('📥 Clone du code') {
            steps {
                echo '=== Récupération du code source ==='
                checkout scm
            }
        }

        stage('📦 Installation des dépendances') {
            steps {
                echo '=== Installation npm ==='
                sh 'npm install || true'
            }
        }

        stage('🔍 SAST - Analyse Semgrep') {
            steps {
                echo '=== Lancement Semgrep ==='
                sh '''
                    mkdir -p ${REPORT_DIR}

                    semgrep \
                        --config ${SEMGREP_RULES} \
                        --json \
                        --output ${REPORT_DIR}/semgrep-report.json \
                        --exclude node_modules \
                        --exclude test \
                        --exclude ".git" \
                        . || true

                    semgrep \
                        --config ${SEMGREP_RULES} \
                        --output ${REPORT_DIR}/semgrep-report.txt \
                        --exclude node_modules \
                        --exclude test \
                        --exclude ".git" \
                        . || true

                    cat ${REPORT_DIR}/semgrep-report.txt || true
                '''
            }
        }

        stage('📊 Résumé des vulnérabilités') {
            steps {
                sh '''
                    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                    echo "    RÉSUMÉ SEMGREP - NodeGoat  "
                    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

                    ERRORS=$(grep -c '"severity": "ERROR"' \
                        ${REPORT_DIR}/semgrep-report.json 2>/dev/null || echo 0)
                    WARNINGS=$(grep -c '"severity": "WARNING"' \
                        ${REPORT_DIR}/semgrep-report.json 2>/dev/null || echo 0)

                    echo "🔴 Critiques  (ERROR)   : $ERRORS"
                    echo "🟡 Moyennes   (WARNING) : $WARNINGS"
                    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
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
                subject: "🔐 Semgrep SAST | ${currentBuild.result} | ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                <html><body style="font-family: Arial;">
                <h2 style="color:#c0392b;">🔐 Rapport SAST — Semgrep</h2>
                <table border="1" cellpadding="10" style="border-collapse:collapse;">
                  <tr><td><b>Projet</b></td><td>OWASP NodeGoat</td></tr>
                  <tr><td><b>Build</b></td><td>#${env.BUILD_NUMBER}</td></tr>
                  <tr><td><b>Statut</b></td><td>${currentBuild.result}</td></tr>
                  <tr><td><b>Durée</b></td><td>${currentBuild.durationString}</td></tr>
                  <tr><td><b>Règles</b></td><td>p/javascript · p/nodejs · p/owasp-top-ten</td></tr>
                </table>
                <br>
                <p>📎 Rapport <b>semgrep-report.txt</b> en pièce jointe</p>
                <a href="${env.BUILD_URL}"
                   style="background:#2980b9;color:white;padding:10px 20px;
                          text-decoration:none;border-radius:4px;">
                  🔗 Voir le Build Jenkins
                </a>
                </body></html>
                """,
                mimeType: 'text/html',
                attachmentsPattern: 'semgrep-reports/semgrep-report.txt'
            )
        }
        success { echo '✅ Pipeline terminé — email envoyé' }
        failure { echo '❌ Pipeline échoué — vérifier les logs' }
    }
}