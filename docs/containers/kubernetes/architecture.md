---
title: Архитектура Kubernetes
---

# 🏛️ Архитектура Kubernetes

**Kubernetes** построен на **клиент-серверной архитектуре**, которая чётко разделяет управляющие компоненты от рабочих.
Это обеспечивает модульность, масштабируемость и высокую отказоустойчивость.

<figure markdown="span">
  ![architecture-overview.png](../assets/notes/admin_panel.png){ width="700" }
  <figcaption>Рисунок 1. Высокоуровневая архитектура Kubernetes</figcaption>
</figure>

!!! abstract "Ключевые принципы"

    *   **Control Plane (Управляющая плоскость):** «Мозг» кластера. Принимает глобальные решения (например, о планировании), а также обнаруживает и реагирует на события в кластере.
    *   **Worker Nodes (Рабочие узлы):** «Руки» кластера. Это машины, которые запускают контейнеризированные приложения.
    *   **`etcd`:** Надёжное key-value хранилище, которое выступает как **«единственный источник правды»** (`single source of truth`) для всего кластера.

---

## 🧠 Control Plane

!!! info "Control Plane — это набор критически важных сервисов, которые управляют состоянием кластера."

В кластерах, развёрнутых с помощью `kubeadm`, эти компоненты обычно запускаются как **статические поды**, манифесты которых лежат в директории `/etc/kubernetes/manifests/` на мастер-узлах.

### 🔌 `kube-apiserver`

**API Server** — это центральный компонент и **единственная точка входа** для всех взаимодействий с кластером.
Он предоставляет RESTful API, через который пользователи, компоненты кластера и внешние инструменты запрашивают и изменяют состояние объектов.

!!! example "Пример популярных опций в `kube-apiserver.yaml`"

    ```yaml
    # /etc/kubernetes/manifests/kube-apiserver.yaml
    apiVersion: v1
    kind: Pod
    # ... metadata ...
    spec:
      containers:
      - command:
        - kube-apiserver
        - --advertise-address=192.168.1.10 # (1)
        - --authorization-mode=Node,RBAC # (2)
        - --client-ca-file=/etc/kubernetes/pki/ca.crt # (3)
        - --enable-admission-plugins=NodeRestriction,PodSecurity # (4)
        - --etcd-servers=https://127.0.0.1:2379 # (5)
        - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
        - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
        - --service-account-issuer=https://kubernetes.default.svc.cluster.local # (6)
        - --service-account-key-file=/etc/kubernetes/pki/sa.pub
        - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
        - --service-cluster-ip-range=10.96.0.0/12 # (7)
        - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
        - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
        image: registry.k8s.io/kube-apiserver:v1.28.0
        name: kube-apiserver
    ```

    1.  **`--advertise-address`**: IP-адрес, который API-сервер анонсирует другим компонентам кластера.
    2.  **`--authorization-mode`**: Включает режимы авторизации. `Node` и `RBAC` — стандартная комбинация.
    3.  **`--client-ca-file`**: CA-сертификат для проверки клиентских сертификатов.
    4.  **`--enable-admission-plugins`**: Включает плагины контроля доступа для обеспечения политик безопасности.
    5.  **`--etcd-servers`**: Адреса серверов `etcd`.
    6.  **`--service-account-issuer`**: URL издателя токенов `ServiceAccount`. Критически важный флаг для работы механизма `BoundServiceAccountTokenVolume`.
    7.  **`--service-cluster-ip-range`**: Диапазон виртуальных IP-адресов для `Services`.

!!! info "Полный список опций"

    Полный и актуальный список всех флагов `kube-apiserver` доступен в [**официальной документации Kubernetes**](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/).

### 🔑 `etcd`

**`etcd`** — это согласованное и высокодоступное key-value хранилище. В `stacked` модели он также запускается как статический под.

!!! danger "etcd — это сердце кластера"
    Потеря данных в `etcd` означает полную потерю состояния кластера. Регулярное создание **бэкапов `etcd`** является критически важной задачей.

!!! example "Пример популярных опций в `etcd.yaml`"
    ```yaml
    # /etc/kubernetes/manifests/etcd.yaml
    apiVersion: v1
    kind: Pod
    # ... metadata ...
    spec:
      containers:
      - command:
        - etcd
        - --name=master-1 # (1)
        - --data-dir=/var/lib/etcd # (2)
        - --listen-client-urls=https://127.0.0.1:2379,https://192.168.1.10:2379 # (3)
        - --advertise-client-urls=https://192.168.1.10:2379 # (4)
        - --listen-peer-urls=https://192.168.1.10:2380 # (5)
        - --initial-advertise-peer-urls=https://192.168.1.10:2380
        - --initial-cluster=master-1=https://192.168.1.10:2380,master-2=... # (6)
        - --cert-file=/etc/kubernetes/pki/etcd/server.crt
        - --key-file=/etc/kubernetes/pki/etcd/server.key
        - --client-cert-auth=true # (7)
        - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
        image: registry.k8s.io/etcd:3.5.9-0
        name: etcd
    ```
    1.  **`--name`**: Уникальное имя узла в кластере `etcd`.
    2.  **`--data-dir`**: Директория на хосте для хранения данных `etcd`.
    3.  **`--listen-client-urls`**: Список адресов, на которых `etcd` слушает запросы от клиентов (например, `kube-apiserver`).
    4.  **`--advertise-client-urls`**: Адрес, который `etcd` сообщает клиентам для подключения.
    5.  **`--listen-peer-urls`**: Адреса для коммуникации между членами кластера `etcd`.
    6.  **`--initial-cluster`**: Список всех узлов кластера `etcd` для первоначальной загрузки.
    7.  **`--client-cert-auth=true`**: Включает обязательную проверку клиентского сертификата.

!!! tip "Резервное копирование `etcd`"

    ```bash
    ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
      --endpoints=https://127.0.0.1:2379 \
      --cacert=/etc/kubernetes/pki/etcd/ca.crt \
      --cert=/etc/kubernetes/pki/etcd/server.crt \
      --key=/etc/kubernetes/pki/etcd/server.key
    ```

!!! info "Полный список опций"

    Полный и актуальный список всех флагов `etcd` доступен в [**официальной документации**](https://etcd.io/docs/v3.5/op-guide/configuration/).

### 📅 `kube-scheduler`

**Планировщик** отвечает за назначение новых подов на подходящие рабочие узлы. Современный способ конфигурации — через файл, а не через множество флагов.

!!! example "Пример манифеста `kube-scheduler.yaml`"
    ```yaml
    # /etc/kubernetes/manifests/kube-scheduler.yaml
    apiVersion: v1
    kind: Pod
    # ... metadata ...
    spec:
      containers:
      - command:
        - kube-scheduler
        - --config=/etc/kubernetes/scheduler-config.yaml # (1)
        - --leader-elect=true # (2)
        - --v=2 # (3)
        image: registry.k8s.io/kube-scheduler:v1.28.0
        name: kube-scheduler
        volumeMounts:
        - name: scheduler-config
          mountPath: /etc/kubernetes/scheduler-config.yaml
          readOnly: true
      volumes:
      - name: scheduler-config
        hostPath:
          path: /etc/kubernetes/scheduler-config.yaml
          type: File
    ```
    1.  **`--config`**: Указывает путь к файлу конфигурации, что является предпочтительным способом настройки.
    2.  **`--leader-elect`**: Включает механизм выбора лидера для обеспечения высокой доступности (HA).
    3.  **`--v=2`**: Устанавливает уровень детализации логов.

!!! example "Пример файла `scheduler-config.yaml`"
    ```yaml
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    leaderElection:
      leaderElect: true
    clientConnection:
      kubeconfig: /etc/kubernetes/scheduler.conf
    profiles:
      - schedulerName: default-scheduler
    ```

!!! info "Полный список опций"

    Полный и актуальный список всех флагов `kube-scheduler` доступен в [**официальной документации Kubernetes**](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/).

### ⚙️ `kube-controller-manager`

Это «диспетчер», который запускает и управляет множеством **контроллеров**.

!!! example "Пример популярных опций в `kube-controller-manager.yaml`"
    ```yaml
    # /etc/kubernetes/manifests/kube-controller-manager.yaml
    apiVersion: v1
    kind: Pod
    # ... metadata ...
    spec:
      containers:
      - command:
        - kube-controller-manager
        - --cluster-cidr=10.244.0.0/16 # (1)
        - --service-cluster-ip-range=10.96.0.0/12 # (2)
        - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt # (3)
        - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
        - --kubeconfig=/etc/kubernetes/controller-manager.conf # (4)
        - --leader-elect=true
        - --service-account-private-key-file=/etc/kubernetes/pki/sa.key # (5)
        - --use-service-account-credentials=true
        image: registry.k8s.io/kube-controller-manager:v1.28.0
        name: kube-controller-manager
    ```
    1.  **`--cluster-cidr`**: Диапазон IP-адресов, из которого CNI-плагин выделяет адреса для подов.
    2.  **`--service-cluster-ip-range`**: Диапазон IP-адресов для `Services`.
    3.  **`--cluster-signing-cert-file`** и **`--cluster-signing-key-file`**: Сертификат и ключ для подписи новых сертификатов в кластере (например, для `kubelet`).
    4.  **`--kubeconfig`**: Путь к файлу для аутентификации на `kube-apiserver`.
    5.  **`--service-account-private-key-file`**: Приватный ключ для подписи токенов `ServiceAccount`.

!!! info "Полный список опций"

    Полный и актуальный список всех флагов `kube-controller-manager` доступен в [**официальной документации Kubernetes**](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/).

### ☁️ `cloud-controller-manager`

Этот компонент запускается как отдельный `Deployment` в кластере и обеспечивает интеграцию с облачным провайдером. Для его активации `kube-apiserver` и `kube-controller-manager` запускаются с флагом `--cloud-provider=external`.

---

## 💪 Worker Nodes

**Рабочие узлы** — это машины, на которых `kubelet` запускает и управляет подами.

### 🏃 `kubelet`

**Kubelet** — это главный агент Kubernetes, работающий на каждом узле. Обычно он устанавливается как сервис `systemd`.

!!! info "Управление `kubelet`"
    ```bash
    # Проверить статус
    sudo systemctl status kubelet

    # Посмотреть логи
    sudo journalctl -u kubelet -f
    ```

Его конфигурация находится в файле `/var/lib/kubelet/config.yaml` и передаётся через аргумент в `systemd unit`.

!!! example "Пример `/var/lib/kubelet/config.yaml`"
    ```yaml
    apiVersion: kubelet.config.k8s.io/v1beta1
    kind: KubeletConfiguration
    cgroupDriver: systemd # (1)
    clusterDNS:
    - 10.96.0.10
    clusterDomain: cluster.local
    evictionHard: # (2)
      imagefs.available: 15%
      memory.available: 100Mi
      nodefs.available: 10%
    serializeImagePulls: false
    ```
    1.  **`cgroupDriver`**: Драйвер `cgroups`. Должен совпадать с драйвером среды выполнения контейнеров (например, `containerd`).
    2.  **`evictionHard`**: Правила "выселения" подов, при которых `kubelet` начнёт удалять поды, если ресурсы узла исчерпаются.

### 🌐 `kube-proxy`

**Kube-proxy** — это сетевой прокси, который работает на каждом узле. Обычно он разворачивается как `DaemonSet` и управляет сетевыми правилами для `Services`. Его конфигурация часто хранится в `ConfigMap`.

!!! example "ConfigMap для `kube-proxy`"
    ```yaml
    # kubectl get configmap kube-proxy -n kube-system -o yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: kube-proxy
      namespace: kube-system
    data:
      config.conf: |
        apiVersion: kubeproxy.config.k8s.io/v1alpha1
        kind: KubeProxyConfiguration
        mode: "ipvs" # (1)
        iptables:
          masqueradeAll: false
    ```
    1.  **`mode`**: Режим работы. `ipvs` является более производительным, чем стандартный `iptables`.

### 📦 Container Runtime (CRI)

Это программное обеспечение, которое отвечает за запуск контейнеров. `containerd` является стандартом де-факто.

!!! info "Взаимодействие с CRI"
    Утилита `crictl` позволяет взаимодействовать с CRI-совместимой средой выполнения для отладки.
    ```bash
    # Список запущенных контейнеров (подов)
    sudo crictl ps

    # Список скачанных образов
    sudo crictl images
    ```

Конфигурация `containerd` находится в `/etc/containerd/config.toml`.

!!! example "Фрагмент `/etc/containerd/config.toml`"
    ```toml
    # ...
    [plugins."io.containerd.grpc.v1.cri"]
      # ...
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
          SystemdCgroup = true # (1)
    # ...
    ```
    1.  **`SystemdCgroup = true`**: Указывает `containerd` использовать `systemd` в качестве `cgroup` драйвера, что должно соответствовать настройке `kubelet`.

---

## 🔄 Поток данных: от `kubectl` до запуска `Pod`

<figure markdown="span">
  ![data-flow.png](../assets/notes/admin_panel.png){ width="700" }
  <figcaption>Рисунок 2. Схема потока данных при создании пода</figcaption>
</figure>

1.  **Пользователь** выполняет команду `kubectl run my-pod --image=nginx`.
2.  **`kubectl`** отправляет REST-запрос на создание объекта `Pod` к **`kube-apiserver`**.
3.  **`kube-apiserver`** валидирует запрос и сохраняет описание пода в **`etcd`**.
4.  **`kube-scheduler`**, который постоянно следит за новыми подами без назначенного узла, обнаруживает `my-pod`.
5.  **`scheduler`** выбирает подходящий `Worker Node` и обновляет объект пода в `etcd`, указывая `spec.nodeName`.
6.  **`kubelet`** на назначенном узле, который следит за подами, назначенными ему, обнаруживает `my-pod`.
7.  **`kubelet`** обращается к **Container Runtime** (например, `containerd`) с инструкцией скачать образ `nginx` и запустить контейнер.
8.  **`kubelet`** постоянно сообщает о состоянии пода обратно в `kube-apiserver`, который обновляет данные в `etcd`.

---

## 🏰 Высокая доступность (High Availability)

Для обеспечения отказоустойчивости Control Plane его компоненты разворачиваются в нескольких экземплярах.

*   **`etcd`**: Разворачивается в виде кластера из 3 или 5 узлов.
*   **`kube-apiserver`**: Размещается на нескольких мастер-узлах, а трафик к ним балансируется.
*   **`kube-scheduler` и `kube-controller-manager`**: Работают в режиме **`leader election`**. В каждый момент времени активна только одна копия, но в случае отказа лидера, другая копия быстро занимает его место.

### 🧱 Модели развёртывания `etcd`

!!! example "Stacked vs External etcd"
    *   **Stacked (Встроенный):** Кластер `etcd` развёрнут на тех же узлах, что и остальные компоненты Control Plane. Проще в настройке, но менее отказоустойчив.
    *   **External (Внешний):** Кластер `etcd` развёрнут на отдельных, выделенных серверах. Более сложен в управлении, но обеспечивает лучшую изоляцию и отказоустойчивость.

---

## ⚖️ API Server Priority and Fairness (APF)

**APF** — это механизм, представленный в Kubernetes для защиты `kube-apiserver` от перегрузки. Он позволяет классифицировать входящие запросы по приоритетам и справедливо распределять ресурсы API-сервера между ними.

!!! success "Зачем нужен APF?"
    Это критически важный механизм для **стабильности Control Plane** под высокой нагрузкой. Он предотвращает ситуацию, когда менее важные, но многочисленные запросы (например, от систем мониторинга) могут «забить» API-сервер и помешать выполнению критически важных операций (например, обновлению подов).