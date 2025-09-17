# Jenkins DevOps Showcase

Репозиторий демонстрирует набор продвинутых Jenkins Pipeline сценариев, которые помогают показать компетенции DevOps-инженера. Каждый пайплайн оформлен отдельным Jenkinsfile и фокусируется на своем направлении.

## Список пайплайнов

1. **01_containerized_build.Jenkinsfile** — Maven сборка в Docker с матричными тестами и архивированием артефактов
2. **02_microservice_matrix.Jenkinsfile** — CI для Node.js микросервиса с матрицей версий и публикацией Docker-образа
3. **03_infrastructure_delivery.Jenkinsfile** — Работа с Terraform: fmt, plan, ручное подтверждение и apply
4. **04_kubernetes_canary.Jenkinsfile** — Канареечный релиз в Kubernetes с параллельной проверкой логов, метрик и синтетики
5. **05_security_compliance.Jenkinsfile** — Комплексная безопасность: SAST, аудит зависимостей, сканирование образа и SonarQube
6. **06_serverless_ci_cd.Jenkinsfile** — Serverless CI/CD для AWS Lambda с нагрузочным тестированием
7. **07_mlops_training.Jenkinsfile** — MLOps цикл: контроль качества данных, обучение, валидация и регистрация модели в MLflow
8. **08_observability_guardrails.Jenkinsfile** — Guardrails для наблюдаемости: promtool, alertmanager, дашборды и синтетика
9. **09_gitops_sync.Jenkinsfile** — GitOps синхронизация с ArgoCD и проверкой дрейфа инфраструктуры
10. **10_chatops_release.Jenkinsfile** — ChatOps релиз с подтверждением через Slack и запуском внешнего deploy-job

## Как использовать

Каждый файл можно положить в отдельный Jenkins job или Multibranch Pipeline. Настройте необходимые credentials (см. `credentialsId` внутри файлов) и обновите команды под ваш стек.

## Дополнительно

В корне репозитория оставлен базовый `Jenkinsfile`, который можно использовать как стартовую точку или заменить одним из представленных сценариев.
