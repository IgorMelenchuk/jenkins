// пайплайн для контроля наблюдаемости сервисов
pipeline {
    agent any
    options {
        timeout(time: 20, unit: 'MINUTES')
        timestamps()
    }
    environment {
        SLACK_CHANNEL = '#observability'
    }
    stages {
        stage('Проверка alertmanager') {
            steps {
                sh 'amtool check-config monitoring/alertmanager.yml'
            }
        }
        stage('Проверка правил prometheus') {
            steps {
                sh 'promtool check rules monitoring/rules/*.yml'
                sh 'promtool test rules monitoring/tests/*.test.yml'
            }
        }
        stage('Валидация дашбордов') {
            steps {
                sh 'gdevvalidate dashboards/**/*.json'
            }
        }
        stage('Прогон синтетики') {
            steps {
                sh 'k6 run tests/observability-smoke.js'
            }
        }
    }
    post {
        success {
            slackSend channel: SLACK_CHANNEL, message: 'Мониторинг в порядке'
        }
        failure {
            slackSend channel: SLACK_CHANNEL, message: 'Нарушены политики наблюдаемости'
        }
    }
}
