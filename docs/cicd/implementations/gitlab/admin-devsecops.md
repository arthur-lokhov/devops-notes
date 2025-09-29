---
title: DevSecOps и Администрирование
---
# 🛡️ DevSecOps и Администрирование

### High Availability (HA) и DR
*   **Ключевые Замечания и SRE/DevOps Фокус:** **Gitaly Clustering** (для HA Git data). **GitLab Geo** (для Disaster Recovery и Multi-Region). **Резервное копирование и Восстановление** (`gitlab-rake`).

### DevSecOps в Пайплайне
*   **Ключевые Замечания и SRE/DevOps Фокус:** **Secrets Management:** Интеграция с **HashiCorp Vault** или использование `secrets:`. **Автосканирование:** Интеграция **SAST, DAST, Dependency Scanning** (использование встроенных шаблонов GitLab). **Container Scanning** (Trivy/Clair).