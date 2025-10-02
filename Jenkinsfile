pipeline {
    agent any

    options {
        skipDefaultCheckout()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        ARTIFACT_NAME = "myapp-${env.BUILD_NUMBER}.jar"
        SLACK_CHANNEL = '#ci-alerts'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Checked out branch: ${env.BRANCH_NAME}"
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh 'mvn test'
                }
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                echo "Artifact archived: ${env.ARTIFACT_NAME}"
            }
        }

        stage('Deploy (Staging)') {
            when {
                branch 'main'
            }
            steps {
                echo "Deploying ${env.ARTIFACT_NAME} to staging environment..."
                // Example: Docker build + push
                // sh 'docker build -t myapp:${BUILD_NUMBER} .'
                // sh 'docker push myrepo/myapp:${BUILD_NUMBER}'
            }
        }
    }

    post {
        success {
            echo "✅ Build #${env.BUILD_NUMBER} succeeded!"
            // slackSend(channel: env.SLACK_CHANNEL, message: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
        failure {
            echo "❌ Build #${env.BUILD_NUMBER} failed!"
            // slackSend(channel: env.SLACK_CHANNEL, message: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
    }
}

