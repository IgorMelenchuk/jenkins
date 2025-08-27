pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Собираем проект...'
            }
        }
        stage('Test') {
            steps {
                echo 'Запускаем тесты...'
            }
        }
        stage('Info') {
            steps {
                echo "Запустились в ветке: ${env.BRANCH_NAME}"
            }
        }
    }
}


