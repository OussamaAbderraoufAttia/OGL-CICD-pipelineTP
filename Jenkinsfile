pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Use bat for Windows; use sh './gradlew build' if on Linux
                bat './gradlew build'
            }
        }
    }
}