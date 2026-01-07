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
                    // Note: Le plugin "Cucumber reports" doit être installé [cite: 13]
                    cucumber buildStatus: 'unstable',
                             fileIncludePattern: '**/*.json',
                             jsonReportDirectory: 'build/test-results/test'
                }
            }
        }
    }
}