// пайплайн для канареечного релиза в kubernetes с проверкой метрик
pipeline {
    agent { label 'k8s-runner' }
    options {
        timeout(time: 35, unit: 'MINUTES')
        timestamps()
    }
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Тег образа для выката')
        string(name: 'NAMESPACE', defaultValue: 'production', description: 'Kubernetes пространство')
    }
    environment {
        RELEASE_NAME = 'service-canary'
        KUBECONFIG = credentials('kubeconfig-prod')
    }
    stages {
        stage('Проверка чарта') {
            steps {
                sh 'helm lint charts/service'
            }
        }
        stage('Подготовка релиза') {
            steps {
                sh "helm upgrade --install ${RELEASE_NAME} charts/service --namespace ${params.NAMESPACE} --set image.tag=${params.IMAGE_TAG} --set deploymentStrategy=canary --set canary.enabled=true --wait --timeout 5m"
            }
        }
        stage('Наблюдение') {
            parallel {
                stage('Проверка логов') {
                    steps {
                        sh "kubectl logs deploy/${RELEASE_NAME} -n ${params.NAMESPACE} --tail=200"
                    }
                }
                stage('Метрики') {
                    steps {
                        sh 'curl -fsSL http://prometheus/api/v1/query?query=rate(http_requests_total[5m])'
                    }
                }
                stage('Синтетические тесты') {
                    steps {
                        sh 'k6 run tests/canary-smoke.js'
                    }
                }
            }
        }
        stage('Переключение трафика') {
            steps {
                sh "helm upgrade ${RELEASE_NAME} charts/service --namespace ${params.NAMESPACE} --set canary.enabled=false --set replicaCount=6 --wait --timeout 5m"
            }
        }
    }
    post {
        success {
            echo 'Канареечный релиз успешно завершен'
        }
        failure {
            sh "helm rollback ${RELEASE_NAME} 0 --namespace ${params.NAMESPACE}"
        }
    }
}
