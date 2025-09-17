// пайплайн безопасной разработки с множеством проверок
pipeline {
    agent any
    options {
        timeout(time: 45, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    tools {
        nodejs 'node20'
    }
    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }
    stages {
        stage('Подготовка зависимостей') {
            steps {
                sh 'npm ci'
            }
        }
        stage('Параллельные проверки') {
            parallel {
                stage('SAST') {
                    steps {
                        sh 'semgrep ci'
                    }
                }
                stage('Dependency Audit') {
                    steps {
                        sh 'npm audit --json > audit.json || true'
                        archiveArtifacts artifacts: 'audit.json', onlyIfSuccessful: false
                    }
                }
                stage('Скан Docker образа') {
                    steps {
                        sh 'trivy image --exit-code 0 --format json --output trivy.json company/app:latest'
                        archiveArtifacts artifacts: 'trivy.json', onlyIfSuccessful: false
                    }
                }
                stage('SonarQube') {
                    steps {
                        withSonarQubeEnv('SonarQube') {
                            sh "sonar-scanner -Dsonar.login=${SONAR_TOKEN}"
                        }
                    }
                }
            }
        }
        stage('Качество политики') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate()
                }
            }
        }
    }
    post {
        success {
            echo 'Продукт прошел комплаенс проверки'
        }
        unstable {
            echo 'Есть замечания безопасности'
        }
    }
}
