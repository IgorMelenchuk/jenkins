// пайплайн для terraform инфраструктуры с ручным подтверждением
pipeline {
    agent any
    options {
        timeout(time: 40, unit: 'MINUTES')
        buildDiscarder(logRotator(daysToKeepStr: '10'))
    }
    triggers {
        pollSCM('H/10 * * * *')
    }
    environment {
        TF_IN_AUTOMATION = 'true'
        BACKEND_BUCKET = 'iac-state-bucket'
    }
    stages {
        stage('Линтинг') {
            steps {
                sh 'terraform fmt -check'
            }
        }
        stage('Инициализация') {
            steps {
                sh 'terraform init -backend-config="bucket=${BACKEND_BUCKET}"'
            }
        }
        stage('Планирование') {
            steps {
                sh 'terraform plan -out=tfplan'
                stash includes: 'tfplan', name: 'tfplan'
            }
        }
        stage('Одобрение') {
            steps {
                script {
                    timeout(time: 15, unit: 'MINUTES') {
                        input message: 'Применить изменения инфраструктуры?' , ok: 'Применить'
                    }
                }
            }
        }
        stage('Применение') {
            when {
                branch 'main'
            }
            steps {
                unstash 'tfplan'
                sh 'terraform apply -input=false tfplan'
            }
        }
    }
    post {
        success {
            echo 'Инфраструктура обновлена'
        }
        failure {
            sh 'terraform show'
        }
        always {
            cleanWs()
        }
    }
}
