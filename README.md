# Лабораторная работа: Kubernetes мессенджер
## Структура репозитория
```
k8s/
├── base/                    # Базовая конфигурация
│   ├── migrations/        # SQL миграции для БД
│   │   ├── users/         # Миграции пользователей (из user-service/migrations)
│   │   └── messages/      # Миграции сообщений (из message-service/migrations)
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── postgres.yaml, postgres-pvc.yaml
│   ├── minio.yaml, minio-secret.yaml, minio-pvc.yaml
│   ├── s3-csi.yaml, csi-secret.yaml  # S3 CSI для загрузки файлов
│   ├── migrate-users-job.yaml, migrate-messages-job.yaml
│   ├── user-service.yaml, message-service.yaml, bff.yaml, frontend.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/               # Dev окружение (latest теги)
│   └── prod/              # Prod окружение (v1.0.0 теги, реплики, ресурсы)
argocd/
├── application.yaml       # Argo CD для dev
└── application-prod.yaml  # Argo CD для prod
docs/runbook.md            # Инструкция запуска
```
## Запуск
Применить dev конфигурацию
```
kubectl apply -k k8s/overlays/dev
```
Или prod
```
kubectl apply -k k8s/overlays/prod
```
## Проверка
```
kubectl get pods -n messager # все pods Running
kubectl get svc -n messager # сервисы доступны
```
Загрузка файла через frontend работает через S3 CSI
Node affinity: postgres/minio на workload=system, прикладные на workload=app
Argo CD: статус Synced/Healthy
## Особенности
```
S3 через CSI: message-service подключает MinIO через ch.ctrox.csi.s3-driver
NodeAffinity:
postgres/minio: required: workload=system
message-service: required: workload=app + preferred: disk=fast
frontend/bff/user-service: required: workload=app
GitOps: Argo CD с automated/prune/selfHeal
Kustomize: base + dev/prod с разными репликами, тегами, ресурсами\
```
