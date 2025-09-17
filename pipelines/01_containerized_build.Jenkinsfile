// использование docker агента для изоляции build окружения
pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        ansiColor('xterm')
        buildDiscarder(logRotator(numToKeepStr: '15'))
    }
    parameters {
        choice(name: 'BUILD_PROFILE', choices: ['dev', 'prod'], description: 'Выбор Maven профиля')
        booleanParam(name: 'RUN_INTEGRATION_TESTS', defaultValue: true, description: 'Запуск интеграционных тестов')
    }
    environment {
        MAVEN_OPTS = '-Dmaven.test.failure.ignore=true'
        BUILD_NUMBER_DISPLAY = "${env.BUILD_NUMBER}-${params.BUILD_PROFILE}"
    }
    stages {
        stage('Подготовка') {
            steps {
                echo "Сборка ${BUILD_NUMBER_DISPLAY}"
                sh 'java -version'
                sh 'mvn -version'
            }
        }
        stage('Статический анализ') {
            steps {
                sh 'mvn -B verify -P${params.BUILD_PROFILE} -DskipTests'
            }
        }
        stage('Юнит тесты') {
            steps {
                sh 'mvn -B test -P${params.BUILD_PROFILE}'
                junit 'target/surefire-reports/*.xml'
            }
        }
        stage('Интеграция') {
            when {
                expression { return params.RUN_INTEGRATION_TESTS }
            }
            steps {
                sh 'mvn -B failsafe:integration-test failsafe:verify -P${params.BUILD_PROFILE}'
                junit 'target/failsafe-reports/*.xml'
            }
        }
        stage('Артефакты') {
            steps {
                sh 'mvn -B package -P${params.BUILD_PROFILE}'
                archiveArtifacts allowEmptyArchive: false, artifacts: 'target/*.jar'
            }
        }
    }
    post {
        success {
            echo "Сборка ${BUILD_NUMBER_DISPLAY} завершена"
        }
        failure {
            echo "Сборка ${BUILD_NUMBER_DISPLAY} провалилась"
        }
    }
}
