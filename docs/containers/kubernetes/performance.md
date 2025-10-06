---
title: Производительность и оптимизация
---

# 🧮 Производительность и оптимизация

Что описать:

*   Подходы к load testing кластера.
*   Pod density на узел.
*   Ограничения control plane (QPS к apiserver).
*   Оптимизация scheduler’а (affinity/anti-affinity, taints/tolerations).
*   Preemption и QoS классы (Guaranteed, Burstable, BestEffort).
*   Kubernetes Pod Overhead.

На что обратить внимание:

*   Как не «убить» кластер большим количеством маленьких подов.
*   Настройка resource requests/limits для прогнозируемого планирования.
*   Мониторинг API server latency.