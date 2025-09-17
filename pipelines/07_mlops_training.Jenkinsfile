// пайплайн mlops с версионированием моделей и проверкой данных
pipeline {
    agent {
        docker {
            image 'python:3.11-slim'
            args '-u root:root'
        }
    }
    options {
        timeout(time: 60, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '8'))
    }
    parameters {
        string(name: 'DATASET_VERSION', defaultValue: '2024-01', description: 'Версия датасета')
    }
    environment {
        MLFLOW_TRACKING_URI = 'http://mlflow:5000'
    }
    stages {
        stage('Установка зависимостей') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }
        stage('Контроль качества данных') {
            steps {
                sh "great_expectations checkpoint run training_data --runtime-kwargs '{\"batch_parameters\":{\"version\":\"${params.DATASET_VERSION}\"}}'"
            }
        }
        stage('Подготовка данных') {
            steps {
                sh 'python pipelines/preprocess.py --dataset ${params.DATASET_VERSION}'
                stash includes: 'artifacts/features.parquet', name: 'features'
            }
        }
        stage('Обучение и оценка') {
            parallel {
                stage('Обучение') {
                    steps {
                        unstash 'features'
                        sh 'python pipelines/train.py --features artifacts/features.parquet --mlflow ${MLFLOW_TRACKING_URI}'
                        stash includes: 'artifacts/model.pkl', name: 'model'
                    }
                }
                stage('Валидация') {
                    steps {
                        unstash 'features'
                        sh 'python pipelines/validate.py --features artifacts/features.parquet'
                        archiveArtifacts allowEmptyArchive: false, artifacts: 'reports/validation/**'
                    }
                }
            }
        }
        stage('Регистрация модели') {
            steps {
                unstash 'model'
                sh 'python pipelines/register.py --model artifacts/model.pkl --stage Staging'
            }
        }
    }
    post {
        success {
            echo 'Модель зарегистрирована в MLflow'
        }
        failure {
            echo 'Тренировка требует расследования'
        }
    }
}
