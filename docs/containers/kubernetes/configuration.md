---
title: Конфигурация
---

# ⚙️ Конфигурация

Что описать:

*   ConfigMap: как env vars, как volume.
*   Secret: base64, монтирование, использование для docker registry creds.
*   ServiceAccount токены и auto-mount.
*   External Secrets, sealed-secrets.
*   Best practices: immutable ConfigMaps, секреты через KMS.

На что обратить внимание:

*   Подводные камни с обновлением ConfigMap (Pod restart нужен).
*   Secrets в etcd → хранить зашифрованными.
*   Secret rotation и аудит.