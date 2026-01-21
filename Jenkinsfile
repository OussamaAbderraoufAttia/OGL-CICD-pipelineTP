pipeline {
    agent any

    // Accessing properties defined in gradle.properties
    environment {
        // These will be read from gradle.properties automatically by Gradle,
        // but we can also reference them here if needed via readProperties
        SONAR_HOST = "http://localhost:9000"
    }

    stages {
        // ========================================
        // Phase 1 : TEST
        // ========================================
        stage('Test') {
            steps {
                echo '========== Phase Test =========='
                bat 'gradlew.bat test'

                echo 'Archivage des resultats...'
                junit '**/build/test-results/test/*.xml'

                cucumber buildStatus: 'SUCCESS',
                         reportTitle: 'Cucumber Test Report',
                         fileIncludePattern: '**/*.json',
                         jsonReportDirectory: 'reports'
            }
        }

        // ========================================
        // Phase 2 : CODE ANALYSIS
        // ========================================
        stage('Code Analysis') {
            steps {
                echo '========== Phase Code Analysis =========='
                withSonarQubeEnv('sonar') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        // The sonar host is pulled from gradle.properties (systemProp.sonar.host.url)
                        bat "gradlew.bat sonar -Dsonar.login=%SONAR_TOKEN%"
                    }
                }
            }
        }

        // ========================================
        // Phase 3 : CODE QUALITY
        // ========================================
        stage('Code Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // ========================================
        // Phase 4 : BUILD
        // ========================================
        stage('Build') {
            steps {
                echo '========== Phase Build =========='
                bat 'gradlew.bat jar'
                bat 'gradlew.bat javadoc'

                archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
                archiveArtifacts artifacts: '**/build/docs/javadoc/**', fingerprint: true
            }
        }

        // ========================================
        // Phase 5 : DEPLOY
        // ========================================
        stage('Deploy') {
            steps {
                echo '========== Phase Deploy =========='
                // Uses 'maven-repo-creds' which should contain:
                // Username: myMavenRepo
                // Password: test0005
                withCredentials([usernamePassword(
                    credentialsId: 'maven-repo-creds',
                    usernameVariable: 'MAVEN_USERNAME',
                    passwordVariable: 'MAVEN_PASSWORD'
                )]) {
                    // Gradle will use url_maven from gradle.properties
                    bat 'gradlew.bat publish'
                }
            }
        }
    }

    // ========================================
    // Phase 6 : NOTIFICATION
    // ========================================
    post {
        success {
            script {
                // Email using variables from gradle.properties or environment
                try {
                    emailext(
                        subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            <h2>Deploiement reussi</h2>
                            <p><b>Projet:</b> ${env.JOB_NAME}</p>
                            <p><b>Status:</b> SUCCESS</p>
                            <p><a href='${env.BUILD_URL}'>Voir le build</a></p>
                        """,
                        to: "li_sraich@esi.dz",
                        replyTo: "lo_attia@esi.dz",
                        mimeType: 'text/html'
                    )
                } catch (err) {
                    echo "Email failed: ${err.toString()}"
                }

                // Slack Notification
                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK_URL')]) {
                    bat """
                        curl -X POST -H "Content-type: application/json" ^
                        --data "{\\"text\\":\\"Deploiement reussi : ${env.JOB_NAME} #${env.BUILD_NUMBER}\\"}" ^
                        %SLACK_WEBHOOK_URL%
                    """
                }
            }
        }

        failure {
            script {
                try {
                    emailext(
                        subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Pipeline failed: ${env.BUILD_URL}",
                        to: "lo_attia@esi.dz"
                    )
                } catch (err) {
                    echo "Email failed"
                }

                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK_URL')]) {
                    bat """
                        curl -X POST -H "Content-type: application/json" ^
                        --data "{\\"text\\":\\"Echec du pipeline : ${env.JOB_NAME} #${env.BUILD_NUMBER}\\"}" ^
                        %SLACK_WEBHOOK_URL%
                    """
                }
            }
        }
    }
}
// modifications