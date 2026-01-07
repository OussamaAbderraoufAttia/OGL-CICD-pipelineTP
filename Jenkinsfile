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
    }
}