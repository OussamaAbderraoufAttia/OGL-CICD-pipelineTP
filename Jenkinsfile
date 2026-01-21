pipeline {
    agent any

    stages {
        // Phase 1 : TEST
        stage('Test') {
            steps {
                echo '========== Phase Test =========='
                // Utilisation de returnStatus pour ne pas bloquer si un test échoue avant le rapport
                bat 'gradlew.bat test'
            }
            post {
                always {
                    echo 'Archivage des resultats...'
                    junit '**/build/test-results/test/*.xml'
                    cucumber buildStatus: 'UNSTABLE',
                             reportTitle: 'Cucumber Test Report',
                             fileIncludePattern: 'example-report.json',
                             jsonReportDirectory: 'reports'
                }
            }
        }

        // Phase 2 : CODE ANALYSIS
        stage('Code Analysis') {
            steps {
                echo '========== Phase Code Analysis =========='
                withSonarQubeEnv('sonar') {
                    // Jenkins injecte automatiquement le token ici
                    bat "gradlew.bat sonar"
                }
            }
        }

        // Phase 3 : CODE QUALITY
        stage('Code Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // Phase 4 : BUILD
        stage('Build') {
            steps {
                echo '========== Phase Build =========='
                bat 'gradlew.bat jar javadoc'
                archiveArtifacts artifacts: '**/build/libs/*.jar, **/build/docs/javadoc/**', fingerprint: true
            }
        }

        // Phase 5 : DEPLOY
        stage('Deploy') {
            steps {
                echo '========== Phase Deploy =========='
                // On s'assure que Jenkins a bien les credentials ID 'maven-repo-creds'
                withCredentials([usernamePassword(
                    credentialsId: 'maven-repo-creds',
                    usernameVariable: 'MAVEN_USERNAME',
                    passwordVariable: 'MAVEN_PASSWORD'
                )]) {
                    bat 'gradlew.bat publish'
                }
            }
        }
    }

    // Phase 6 : NOTIFICATION
    post {
        success {
            script {
                emailext(
                    subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "<h2>Deploiement reussi</h2><p>Projet: ${env.JOB_NAME}</p><p><a href='${env.BUILD_URL}'>Voir le build</a></p>",
                    to: "li_sraich@esi.dz",
                    mimeType: 'text/html'
                )

                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_URL')]) {
                    bat "curl -X POST -H \"Content-type: application/json\" --data \"{\\\"text\\\":\\\"✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}\\\"}\" %SLACK_URL%"
                }
            }
        }
        failure {
            script {
                emailext(
                    subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "Le pipeline a echoue. Voir ici: ${env.BUILD_URL}",
                    to: "lo_attia@esi.dz"
                )

                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_URL')]) {
                    bat "curl -X POST -H \"Content-type: application/json\" --data \"{\\\"text\\\":\\\"❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}\\\"}\" %SLACK_URL%"
                }
            }
        }
    }
}
// updates