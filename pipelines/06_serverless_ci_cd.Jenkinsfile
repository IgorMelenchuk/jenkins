// пайплайн для serverless приложения с прогоном нагрузочных тестов
pipeline {
    agent any
    options {
        timeout(time: 30, unit: 'MINUTES')
        durabilityHint('PERFORMANCE_OPTIMIZED')
    }
    environment {
        AWS_DEFAULT_REGION = 'eu-central-1'
        AWS_ACCESS_KEY_ID = credentials('aws-access')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret')
    }
    stages {
        stage('Установка CLI') {
            steps {
                sh 'npm install -g serverless artillery'
            }
        }
        stage('Сборка') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }
        stage('Тесты') {
            steps {
                sh 'npm test'
            }
        }
        stage('Деплой стека') {
            steps {
                sh 'serverless deploy --stage prod --conceal'
            }
        }
        stage('Нагрузочное тестирование') {
            steps {
                sh 'artillery run tests/load.yml'
                archiveArtifacts artifacts: 'tests/results/**/*', allowEmptyArchive: true
            }
        }
        stage('Откат') {
            when {
                expression { currentBuild.currentResult != 'SUCCESS' }
            }
            steps {
                sh 'serverless rollback'
            }
        }
    }
    post {
        success {
            echo 'Лямбда функции обновлены'
        }
        failure {
            echo 'Деплой прерван'
        }
    }
}
