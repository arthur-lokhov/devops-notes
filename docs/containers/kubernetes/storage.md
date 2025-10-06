---
title: Хранилище
---

# 💾 Хранилище

Что описать:

*   PersistentVolume (PV) и PersistentVolumeClaim (PVC).
*   StorageClass и динамическое выделение.
*   CSI (Container Storage Interface).
*   Access modes: ReadWriteOnce, ReadOnlyMany, ReadWriteMany.
*   ReclaimPolicy: Retain/Delete/Recycle.
*   VolumeSnapshots и восстановление.

На что обратить внимание:

*   Как PVC «биндится» к PV.
*   Когда использовать hostPath (только тесты).
*   Ограничения RWX/RWO.
*   StorageClass параметры (provisioner, reclaimPolicy).