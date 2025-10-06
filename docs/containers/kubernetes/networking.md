---
title: Сеть
---

# 🌐 Сеть

Что описать:

*   CNI: назначение, примеры (Flannel, Calico, Cilium).
*   kube-proxy: iptables vs ipvs.
*   Cluster DNS (CoreDNS).
*   Service types: ClusterIP, NodePort, LoadBalancer, ExternalName.
*   Ingress и Ingress Controller.
*   NetworkPolicy: декларативное ограничение трафика.
*   External/Egress Network Policy.

На что обратить внимание:

*   Разница L3/L4 (Service) и L7 (Ingress).
*   EndpointSlice и масштабирование.
*   NetworkPolicy: по умолчанию «разрешено всё», best practice → «deny all, allow needed».
*   Траблшутинг сети: DNS, kube-proxy, CNI pods.
*   Преимущества eBPF в CNI плагинах (например, Cilium) по сравнению с iptables/ipvs.
*   Поддержка IPv6 и Dual-stack в кластерах.