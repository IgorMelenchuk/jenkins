// пайплайн chatops релиза с подтверждением из slack
pipeline {
    agent any
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    parameters {
        choice(name: 'TARGET_ENV', choices: ['staging', 'production'], description: 'Среда для релиза')
    }
    environment {
        SLACK_CHANNEL = '#deployments'
    }
    stages {
        stage('Подготовка релиза') {
            steps {
                sh 'git fetch --tags'
                sh 'git describe --tags --abbrev=0 > VERSION'
                archiveArtifacts artifacts: 'VERSION', onlyIfSuccessful: true
            }
        }
        stage('Согласование') {
            steps {
                script {
                    slackSend channel: SLACK_CHANNEL, message: "Запрошен релиз в ${params.TARGET_ENV}"
                    timeout(time: 20, unit: 'MINUTES') {
                        input message: "Одобрить релиз в ${params.TARGET_ENV}?", ok: 'Одобряю'
                    }
                }
            }
        }
        stage('Деплой') {
            steps {
                build job: 'deploy-service', parameters: [string(name: 'ENV', value: params.TARGET_ENV)]
            }
        }
        stage('Проверка') {
            steps {
                sh "./scripts/verify.sh ${params.TARGET_ENV}"
            }
        }
    }
    post {
        success {
            slackSend channel: SLACK_CHANNEL, message: "Релиз ${params.TARGET_ENV} завершен"
        }
        failure {
            slackSend channel: SLACK_CHANNEL, message: "Релиз ${params.TARGET_ENV} упал"
        }
    }
}
