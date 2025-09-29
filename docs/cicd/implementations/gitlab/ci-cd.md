---
title: GitLab CI/CD
---
# 🦊 GitLab CI/CD

### Управление GitLab Runners
*   **Ключевые Замечания и SRE/DevOps Фокус:** **Kubernetes Executor** как лучшая практика (для масштабирования и изоляции). **Docker Executor и DIND:** Риски и альтернативы (Kaniko). **Изоляция:** Правильное использование `tags:` и ограниченные права Runner'ов.

### Пайплайн как Код (`.gitlab-ci.yml`)
*   **Ключевые Замечания и SRE/DevOps Фокус:** **Оптимизация:** Настройка `cache:` и `artifacts:` для минимизации времени сборки. **Dynamic Pipelines:** `rules:`, `needs:`, **Parent-Child Pipelines** для сложных проектов.

### Паттерны Развертывания
*   **Ключевые Замечания и SRE/DevOps Фокус:** Использование `environment:` и **Protected Environments**. Реализация **Blue/Green, Canary** стратегий (связь с Kubernetes). Обязательная настройка **Rollback** (как SRE-метрика).

### GitLab Agent for Kubernetes (GitOps)
*   **Ключевые Замечания и SRE/DevOps Фокус:** Установка и использование агента для безопасного развертывания в кластеры K8s без необходимости пробрасывать Kubeconfig в Runner.