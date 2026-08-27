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
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.sources=.
                    '''
                }
            }
        }
    }
}
