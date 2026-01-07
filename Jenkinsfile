pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                // 1. Lancement des tests unitaires via le wrapper Gradle
                bat './gradlew test'
            }
            post {
                always {
                    // Archivage des résultats JUnit (Standard)
                    junit '**/build/test-results/test/*.xml'

                    // Génération du rapport Cucumber
                    cucumber buildStatus: 'unstable',
                             fileIncludePattern: 'example-report.json',
                             jsonReportDirectory: 'reports'
                }
            }
        }
        stage('Code Analysis') {
                     steps {
                         // Utilisation du nom du serveur configuré dans Jenkins (ex: 'sonar')
                         withSonarQubeEnv('sonar') {
                             bat './gradlew sonar'
                         }
                     }
                 }

                 stage("Code Quality") {
                     steps {
                         // Attend le retour du Webhook de SonarQube pour valider la Quality Gate
                         timeout(time: 1, unit: 'HOURS') {
                             waitForQualityGate abortPipeline: true
                         }
                     }
                 }
    }
}