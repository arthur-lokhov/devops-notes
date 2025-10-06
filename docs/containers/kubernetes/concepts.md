---
title: Основные концепции
---

# 🧩 Основные концепции (строительные блоки)

Что описать:

*   Pod: структура, lifecycle, initContainers, sidecars.
*   Labels, Selectors, Annotations — зачем нужны.
*   Namespaces, их назначение, resource quotas, limit ranges.
*   OwnerReferences и garbage collection.
*   ServiceAccounts и связь с подами.

На что обратить внимание:

*   Pod как минимальная единица планирования (не контейнер).
*   Как GC удаляет дочерние ресурсы.
*   Правильное использование namespaces (dev/prod isolation, multi-tenancy).
*   Разница annotations vs labels.
