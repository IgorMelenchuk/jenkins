pipeline {
    agent any
    options {
        timeout(time: 20, unit: 'MINUTES')
        retry(2)
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    stages {
        stage('Info') {
            steps {
                echo "Запустились в ветке: ${env.BRANCH_NAME}"
            }
        }
        stage('Build') {
            steps {
                echo 'Сборка проекта...'
            }
        }
        stage('Test') {
            steps {
                echo 'Тестирование проекта...'
            }
        }
    }
}
