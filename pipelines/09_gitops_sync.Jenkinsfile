// пайплайн gitops синхронизации с использованием argocd
pipeline {
    agent any
    options {
        timeout(time: 25, unit: 'MINUTES')
    }
    environment {
        ARGOCD_AUTH_TOKEN = credentials('argocd-token')
        ARGOCD_SERVER = 'argocd.example.com'
    }
    stages {
        stage('Генерация манифестов') {
            steps {
                sh 'kustomize build environments/prod > dist.yaml'
                archiveArtifacts artifacts: 'dist.yaml', onlyIfSuccessful: true
            }
        }
        stage('Проверка дрейфа') {
            steps {
                sh "argocd app diff platform --server ${ARGOCD_SERVER} --auth-token ${ARGOCD_AUTH_TOKEN} --grpc-web || true"
            }
        }
        stage('Синхронизация') {
            steps {
                sh "argocd app sync platform --server ${ARGOCD_SERVER} --auth-token ${ARGOCD_AUTH_TOKEN} --grpc-web --timeout 300"
            }
        }
        stage('Проверка статуса') {
            steps {
                sh "argocd app wait platform --server ${ARGOCD_SERVER} --auth-token ${ARGOCD_AUTH_TOKEN} --grpc-web --timeout 300 --health"
            }
        }
    }
    post {
        success {
            echo 'Кластер синхронизирован с git'
        }
        failure {
            echo 'Проверить argocd события'
        }
    }
}
