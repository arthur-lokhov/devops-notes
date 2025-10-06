---
title: Документация по Kubernetes (структура/roadmap)
---

# 📘 Документация по Kubernetes (структура/roadmap)

## 1. 🏛️ Архитектура

**Что описать:**

*   Общая модель: `Control Plane` vs `Worker Nodes`.
*   Компоненты `Control Plane`: `kube-apiserver`, `etcd`, `kube-scheduler`, `kube-controller-manager`, `cloud-controller-manager`.
*   Компоненты `Worker Node`: `kubelet`, `kube-proxy`, `CRI` (`containerd`, `CRI-O`, `Docker-shim`), `CNI` плагины.
*   Поток данных: от `kubectl` до запуска `Pod`.
*   **HA-модели**: `stacked etcd` vs `external etcd`.
*   `Leader election`, `watchers`, `reconciliation loop`.
*   **API Server Priority and Fairness (APF)**

**На что обратить внимание:**

*   Где хранится состояние (`etcd`) и что значит **«источник правды»**.
*   Как работает `reconciliation loop` (**desired** vs **actual state**).
*   Ограничения `etcd` (производительность, бэкапы, сетевые задержки).
*   Что ломается при отказе каждого компонента.
*   Механизм `API Server Priority and Fairness (APF)` для стабильности `Control Plane` под нагрузкой.

---

## 2. 🧩 Основные концепции (строительные блоки)

**Что описать:**

*   `Pod`: структура, `lifecycle`, `initContainers`, `sidecars`.
*   `Labels`, `Selectors`, `Annotations` — зачем нужны.
*   `Namespaces`, их назначение, `resource quotas`, `limit ranges`.
*   `OwnerReferences` и `garbage collection`.
*   `ServiceAccounts` и связь с подами.

**На что обратить внимание:**

*   `Pod` как **минимальная единица планирования** (не контейнер).
*   Как `GC` удаляет дочерние ресурсы.
*   Правильное использование `namespaces` (`dev/prod isolation`, `multi-tenancy`).
*   Разница `annotations` vs `labels`.

---

## 3. 🚀 Рабочие нагрузки (Workloads)

**Что описать:**

*   `Deployment`: реплики, стратегии обновления (`rolling update`, `recreate`, `canary`).
*   `StatefulSet`: уникальные идентификаторы, стабильные сетевые имена, `PVC templates`.
*   `DaemonSet`: агенты/мониторинг.
*   `Jobs` и `CronJobs`: одноразовые задачи, расписания.
*   `PodDisruptionBudgets` и их роль в **HA**.

**На что обратить внимание:**

*   Как сделать `rollback/undo` в `Deployment`.
*   Где лучше использовать `StatefulSet`, где `Deployment`.
*   `DaemonSet` + `tolerations`/`node selectors`.
*   Ограничения `CronJobs` (`backoff`, `missed runs`).

---

## 4. 🌐 Сеть

**Что описать:**

*   `CNI`: назначение, примеры (`Flannel`, `Calico`, `Cilium`).
*   `kube-proxy`: `iptables` vs `ipvs`.
*   `Cluster DNS` (`CoreDNS`).
*   `Service types`: `ClusterIP`, `NodePort`, `LoadBalancer`, `ExternalName`.
*   `Ingress` и `Ingress Controller`.
*   `NetworkPolicy`: декларативное ограничение трафика.
*   `External/Egress Network Policy`.

**На что обратить внимание:**

*   Разница `L3`/`L4` (`Service`) и `L7` (`Ingress`).
*   `EndpointSlice` и масштабирование.
*   `NetworkPolicy`: по умолчанию «разрешено всё», **best practice** → «deny all, allow needed».
*   Траблшутинг сети: `DNS`, `kube-proxy`, `CNI pods`.
*   Преимущества `eBPF` в `CNI` плагинах (например, `Cilium`) по сравнению с `iptables`/`ipvs`.
*   Поддержка `IPv6` и `Dual-stack` в кластерах. 

---

## 5. 💾 Хранилище

**Что описать:**

*   `PersistentVolume` (`PV`) и `PersistentVolumeClaim` (`PVC`).
*   `StorageClass` и динамическое выделение.
*   `CSI` (`Container Storage Interface`).
*   `Access modes`: `ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany`.
*   `ReclaimPolicy`: `Retain`/`Delete`/`Recycle`.
*   `VolumeSnapshots` и восстановление.

**На что обратить внимание:**

*   Как `PVC` «биндится» к `PV`.
*   Когда использовать `hostPath` (только тесты).
*   Ограничения `RWX`/`RWO`.
*   `StorageClass` параметры (`provisioner`, `reclaimPolicy`).

---

## 6. ⚙️ Конфигурация

**Что описать:**

*   `ConfigMap`: как `env vars`, как `volume`.
*   `Secret`: `base64`, монтирование, использование для `docker registry creds`.
*   `ServiceAccount` токены и `auto-mount`.
*   `External Secrets`, `sealed-secrets`.
*   **Best practices**: `immutable ConfigMaps`, секреты через `KMS`.

**На что обратить внимание:**

*   Подводные камни с обновлением `ConfigMap` (`Pod restart` нужен).
*   `Secrets` в `etcd` → хранить зашифрованными.
*   `Secret rotation` и аудит.

---

## 7. 🛡️ Безопасность

**Что описать:**

*   `RBAC`: `Roles`, `ClusterRoles`, `Bindings`.
*   `Pod Security Standards` (`restricted`/`baseline`/`privileged`).
*   `Admission Controllers` (встроенные + динамические).
*   `NetworkPolicy` как часть `security`.
*   Инструменты: `OPA/Gatekeeper`, `Kyverno`.
*   `CIS Benchmarks`, `kube-bench`.

**На что обратить внимание:**

*   **Principle of Least Privilege**.
*   `Drop capabilities`, `runAsNonRoot`, `seccomp`, `AppArmor`.
*   `Secrets`: не хранить в `git` в чистом виде.
*   **Common misconfigs** (`privileged pods`, `hostPath`, `wide RBAC`).

---

## 8. 🛠️ Инструменты

**Что описать:**

*   `kubectl` (базовые и продвинутые команды).
*   `kubeadm` (создание, `upgrade`).
*   `kind`, `minikube`, `k3d` (локальные кластеры).
*   `eksctl`, `gcloud`, `aks CLI` — управляемые сервисы.
*   `krew` + плагины (`neat`, `debug`, `ctx`, `ns`).

**На что обратить внимание:**

*   Отличие локальных тулов vs `production`.
*   `Node lifecycle`: `cordon`/`drain`/`uncordon`.
*   `Cluster upgrade flow`.

---

## 9. 📦 Управление пакетами (Helm)

**Что описать:**

*   `Helm charts`: структура, `templates`, `values.yaml`.
*   `Lifecycle`: `install`, `upgrade`, `rollback`, `history`.
*   `Chart repository` и управление зависимостями.
*   `Security`: проверка и подпись чартов.

**На что обратить внимание:**

*   Разница `helm template` vs `helm install`.
*   Как `override`-ить `values`.
*   `Chart testing`, `lint`.

---

## 10. 🎨 Кастомизация манифестов (Kustomize)

**Что описать:**

*   `Base` и `overlay`.
*   `Patches` (`strategic merge`, `JSON patch`).
*   `Variants`: `dev`/`staging`/`prod`.
*   Интеграция с `kubectl`.

**На что обратить внимание:**

*   Когда `Kustomize` vs `Helm`.
*   Комбинирование: `helm template` → `kustomize overlay`.
*   Поддержка `GitOps`.

---

## 11. 📊 Наблюдаемость (Observability)

**Что описать:**

*   `Metrics`: `kubelet`, `kube-state-metrics`, `Prometheus`.
*   `Logs`: `EFK` (`Elasticsearch`/`Fluentd`/`Kibana`) или `Loki`.
*   `Tracing`: `OpenTelemetry`, `Jaeger`.
*   `Alerts`: `Prometheus Alertmanager`.
*   `Dashboards`: `Grafana`.

**На что обратить внимание:**

*   `SLI`/`SLO` vs `SLA`.
*   `Alert fatigue` и **best practices** (`multi-level alerts`).
*   `Long-term storage` (`Thanos`, `Cortex`).

---

## 12. 🚀 CI/CD и GitOps

**Что описать:**

*   `GitOps` концепция.
*   `ArgoCD`/`Flux`: приложения из `Git`.
*   `Progressive delivery`: `canary`, `blue-green`, `A/B`.
*   `CI/CD pipelines`: `build image`, `push`, `deploy`.

**На что обратить внимание:**

*   **Immutable инфраструктура**.
*   `Rollback` стратегии.
*   Интеграция с `security scanning`.

---

## 13. ⚡ Production readiness & SRE-практики

**Что описать:**

*   `Disaster Recovery`: бэкапы `etcd`, `PV`, `restore`-процедуры.
*   `Upgrade strategy`: `minor`/`major`, `zero-downtime`.
*   `Incident response`: `runbooks`, `playbooks`.
*   `Capacity planning` & `autoscaling`: `HPA`, `VPA`, `Cluster Autoscaler`.
*   `Multi-cluster` и `multi-region deployment`.

**На что обратить внимание:**

*   **Test your DR** (не только бэкап, но и `restore`).
*   `Canary release` + `SLOs` → `rollback triggers`.
*   **Cost optimization** (`spot`-инстансы, `bin packing`).

---

## 14. 🧩 Advanced / Extensibility

**Что описать:**

*   `CRD` и `Custom Controllers`.
*   `Operators`: принципы, `kubebuilder`, `operator-sdk`.
*   `Admission Webhooks`.
*   `API Aggregation`.
*   `Kubernetes Gateway API` как замена `Ingress`.

**На что обратить внимание:**

*   Контроллер как `reconciliation loop`.
*   Пример кастомного `CRD`: «MySQLCluster» → управляет `StatefulSet`.
*   Как писать надёжный оператор.

---

## 15. 📑 Приложения: кейсы и best practices

**Что описать:**

*   `Stateless web-app`.
*   `Stateful DB`.
*   `Batch jobs`.
*   `Monitoring`/`Logging stack`.
*   `Multi-tenant cluster`.

**На что обратить внимание:**

*   Ошибки конфигурации (`latest image`, `no limits`, `wide RBAC`).
*   Паттерны Kubernetes (`Sidecar`, `Ambassador`, `Adapter`).
*   Оптимизация стоимости.

---

## 16. 🌍 Экосистема и CNCF-стек

**Что описать:**

*   Как `Kubernetes` встроен в экосистему `CNCF`.
*   Связка с `Service Mesh` (`Istio`, `Linkerd`, `Consul`).
*   `Knative` (`serverless` поверх `K8s`).
*   `Open Policy Agent` (`OPA`) не только для `security`, но и как часть `governance`.
*   `Crossplane` для управления ресурсами вне `Kubernetes` (`cloud`, `DB`).
*   `Cluster Federation (KubeFed)` и `Multi-cluster Service API`.

**На что обратить внимание:**

*   Где `Kubernetes` решает задачу, а где лучше взять инструмент поверх.
*   Как не «прикручивать всё подряд» → **keep it simple**.

---

## 17. 🔄 Сетевая сложность и сервис-меш

**Что описать:**

*   `Service Mesh`: `sidecar-proxy`, `data plane` vs `control plane`.
*   `Istio`/`Linkerd`/`Cilium Service Mesh`.
*   `mTLS`, распределённый трейсинг через `mesh`.
*   `Policy enforcement` (`RBAC` на уровне сети).
*   Нагрузка на сеть и `overhead` от `mesh`.

**На что обратить внимание:**

*   Когда `mesh` оправдан, а когда нет.
*   Как `mesh` влияет на `latency`, отладку и наблюдаемость.

---

## 18. 🧮 Производительность и оптимизация

**Что описать:**

*   Подходы к `load testing` кластера.
*   `Pod density` на узел.
*   Ограничения `control plane` (`QPS` к `apiserver`).
*   Оптимизация `scheduler`’а (`affinity`/`anti-affinity`, `taints`/`tolerations`).
*   `Preemption` и `QoS` классы (`Guaranteed`, `Burstable`, `BestEffort`).
*   `Kubernetes Pod Overhead`.

**На что обратить внимание:**

*   Как не «убить» кластер большим количеством маленьких `pod`'''ов.
*   Настройка `resource requests`/`limits` для прогнозируемого планирования.
*   Мониторинг `API server latency`.

---

## 19. 📜 Compliance & Audit

**Что описать:**

*   `Audit logs` Kubernetes.
*   Интеграция с `SIEM`.
*   Регуляторные требования: `PCI DSS`, `GDPR`, `HIPAA`.
*   `Open Policy Agent` + `GitOps` = **enforce compliance**.
*   `Falco` (`runtime security`, `audit events`).

**На что обратить внимание:**

*   Где и как хранить `audit logs`.
*   Подводные камни с `privileged containers` для `compliance`.
*   **Automation** → «policy as code».

---

## 20. 🧰 Platform Engineering / Internal Developer Platform

**Что описать:**

*   Как построить «PaaS внутри компании» на `Kubernetes`.
*   `Backstage` (`developer portal`).
*   `Abstractions`: `Application CRD` vs `raw manifests`.
*   `Self-service` для команд.
*   `Golden Paths` (**best practice** манифесты).

**На что обратить внимание:**

*   Где граница между «удобно» и «overengineering».
*   Как дать разработчикам простоту, а `DevOps` оставить контроль.
*   Опыт больших компаний (`Spotify`/`Netflix`/`Shopify`).

---

## 21. 🔬 Debugging & Chaos Engineering

**Что описать:**

*   Инструменты отладки: `kubectl debug`, `ephemeral containers`, `strace`, `tcpdump`.
*   `Chaos Mesh`/`LitmusChaos`: ввод отказов, тестирование `resiliency`.
*   Отладка проблем: `CrashLoopBackoff`, `ImagePullBackOff`, сетевые проблемы.
*   `Debugging in production` (без `downtime`).

**На что обратить внимание:**

*   Логика «simulate failure before it happens».
*   Как тренировать команду на `chaos game days`.
*   `Runbooks` как часть `SRE`-культуры.

---

## 22. 💸 Финансовая оптимизация (FinOps)

**Что описать:**

*   `Cluster Autoscaler` + `Karpenter` (`AWS`).
*   `Spot-инстансы` и `preemptible VMs`.
*   `Request`/`limit tuning` для `cost-efficiency`.
*   `Billing per namespace`/`project`.
*   `OpenCost`/`Kubecost`.

**На что обратить внимание:**

*   `Tradeoff` между `cost` и `reliability`.
*   Как объяснять бизнесу «почему `Kubernetes` дорогой».
*   Оптимизация `CI/CD pipeline workloads`.