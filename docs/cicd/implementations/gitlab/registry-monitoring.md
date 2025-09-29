---
title: Registry & Мониторинг
---
# 📦 Registry & Мониторинг

### GitLab Container Registry
*   **Ключевые Замечания и SRE/DevOps Фокус:** Настройка квот (Quotas). **Image Cleanup Policy:** Автоматическая очистка старых образов (критично для SRE).

### Мониторинг GitLab (SRE)
*   **Ключевые Замечания и SRE/DevOps Фокус:** **Сбор Метрик:** Интеграция с **Prometheus/Grafana** (мониторинг Gitaly, Sidekiq, Rails). **Сбор Логов:** Централизованное логирование GitLab (ELK/Loki).

### Мониторинг CI/CD
*   **Ключевые Замечания и SRE/DevOps Фокус:** **SLO/SLA** для пайплайнов: мониторинг **времени выполнения** и **коэффициента успешности** Runner'ов. **Troubleshooting** Job'ов.