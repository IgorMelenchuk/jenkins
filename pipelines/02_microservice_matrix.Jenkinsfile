// матричное тестирование микросервиса с упаковкой в docker
pipeline {
    agent any
    options {
        timeout(time: 25, unit: 'MINUTES')
        parallelsAlwaysFailFast()
        disableConcurrentBuilds()
    }
    environment {
        REGISTRY = 'registry.example.com/company/service'
    }
    stages {
        stage('Получение кода') {
            steps {
                checkout scm
            }
        }
        stage('Матричные тесты') {
            matrix {
                axes {
                    axis {
                        name 'NODE_VERSION'
                        values '18', '20'
                    }
                }
                agent {
                    docker {
                        image "node:${NODE_VERSION}-alpine"
                        args '-u root:root'
                    }
                }
                stages {
                    stage('Установка зависимостей') {
                        steps {
                            sh 'npm ci'
                        }
                    }
                    stage('Запуск тестов') {
                        steps {
                            sh 'npm test -- --ci --reporters=jest-junit'
                            junit 'reports/junit.xml'
                        }
                    }
                }
                post {
                    always {
                        archiveArtifacts allowEmptyArchive: true, artifacts: 'reports/**/*'
                    }
                }
            }
        }
        stage('Статический анализ') {
            steps {
                sh 'npm run lint'
            }
        }
        stage('Сборка образа') {
            steps {
                script {
                    docker.withRegistry("https://${env.REGISTRY}", 'registry-credentials') {
                        def app = docker.build("${env.REGISTRY}:${env.BUILD_NUMBER}")
                        app.push()
                        app.push('latest')
                    }
                }
            }
        }
        stage('Проверка чарта') {
            steps {
                sh 'helm lint charts/service'
            }
        }
    }
    post {
        success {
            echo 'Микросервис готов к деплою'
        }
        unsuccessful {
            echo 'Пайплайн остановлен до фикса'
        }
    }
}
