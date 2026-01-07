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
                    // 2. Archivage des résultats des tests unitaires pour Jenkins
                    junit 'build/test-results/test/*.xml'

                    // 3. Génération du rapport visuel Cucumber
                    cucumber buildStatus: 'unstable',
                             fileIncludePattern: '**/*.json',
                             jsonReportDirectory: 'json:reports/'
                }
            }
        }
    }
}