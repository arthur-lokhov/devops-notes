---
title: Безопасность
---

# 🛡 Безопасность

Что описать:

*   RBAC: Roles, ClusterRoles, Bindings.
*   Pod Security Standards (restricted/baseline/privileged).
*   Admission Controllers (встроенные + динамические).
*   NetworkPolicy как часть security.
*   Инструменты: OPA/Gatekeeper, Kyverno.
*   CIS Benchmarks, kube-bench.

На что обратить внимание:

*   Principle of Least Privilege.
*   Drop capabilities, runAsNonRoot, seccomp, AppArmor.
*   Secrets: не хранить в git в чистом виде.
*   Common misconfigs (privileged pods, hostPath, wide RBAC).