---
title: Production readiness & SRE-практики
---

# ⚡️ Production readiness & SRE-практики

Что описать:

*   Disaster Recovery: бэкапы etcd, PV, restore-процедуры.
*   Upgrade strategy: minor/major, zero-downtime.
*   Incident response: runbooks, playbooks.
*   Capacity planning & autoscaling: HPA, VPA, Cluster Autoscaler.
*   Multi-cluster и multi-region deployment.

На что обратить внимание:

*   Test your DR (не только бэкап, но и restore).
*   Canary release + SLOs → rollback triggers.
*   Cost optimization (spot-инстансы, bin packing).