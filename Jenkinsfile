pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'jenkins-securise-demo'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Analyse statique - SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    script {
                        def scannerHome = tool 'SonarScanner'
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.sources=."
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Vérification déploiement') {
            steps {
                sh 'curl -s -o /dev/null -w "Code HTTP : %{http_code}\\n" http://juice-shop:3000'
            }
        }
        stage('Scan dynamique - OWASP ZAP') {
            steps {
                sh '''
                    docker run --rm --network projetdocker_devsecops-net \
                    -v /var/lib/docker/volumes/projetdocker_jenkins_home/_data/workspace/pipeline-cicd-securise:/zap/wrk/:rw \
                    -u root \
                    ghcr.io/zaproxy/zaproxy:stable \
                    zap-baseline.py -t http://juice-shop:3000 -r zap-report.html -d || true
                    echo "=== Contenu complet du workspace apres scan ==="
                    find . -type f -newer Jenkinsfile
                    echo "=== Fin du diagnostic ==="
                '''
            }
        }
        stage('Publication des rapports') {
            steps {
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'zap-report.html',
                    reportName: 'Rapport-OWASP-ZAP'
                ])
            }
        }
    }
}
