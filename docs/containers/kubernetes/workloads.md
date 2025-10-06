---
title: Рабочие нагрузки (Workloads)
---

# 🚀 Рабочие нагрузки (Workloads)

Что описать:

*   Deployment: реплики, стратегии обновления (rolling update, recreate, canary).
*   StatefulSet: уникальные идентификаторы, стабильные сетевые имена, PVC templates.
*   DaemonSet: агенты/мониторинг.
*   Jobs и CronJobs: одноразовые задачи, расписания.
*   PodDisruptionBudgets и их роль в HA.

На что обратить внимание:

*   Как сделать rollback/undo в Deployment.
*   Где лучше использовать StatefulSet, где Deployment.
*   DaemonSet + tolerations/node selectors.
*   Ограничения CronJobs (backoff, missed runs).