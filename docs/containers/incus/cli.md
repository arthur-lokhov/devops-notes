---
title: Команды CLI (incus)
hide:
  - toc
---

# 🛠️ Команды CLI (incus)

**Incus** предоставляет единый и интуитивно понятный интерфейс командной строки (`incus`) для управления всеми аспектами жизненного цикла контейнеров и виртуальных машин.

!!! abstract "Ключевое отличие от LXC"
    Вместо множества отдельных утилит (`lxc-create`, `lxc-start` и т.д.), Incus использует единую команду с субкомандами, что делает его похожим на `docker` или `git`.
    
    `incus <объект> <действие>`

---

## 🚀 Основные команды

### `incus launch`

Создаёт и запускает новый экземпляр (контейнер или ВМ) из образа.

```bash
# Запустить контейнер Ubuntu 22.04 с именем 'my-container'
incus launch images:ubuntu/22.04 my-container

# Запустить виртуальную машину
incus launch images:ubuntu/22.04 my-vm --vm

# Запустить контейнер с определенным профилем
incus launch images:alpine/edge my-alpine -p default -p my-profile
```

### `incus list`

Показывает список всех экземпляров (контейнеров и ВМ).

```bash
# Показать все экземпляры
incus list

# Отфильтровать по имени
incus list my-container

# Показать в формате JSON
incus list --format json
```

### `incus exec`

Выполняет команду внутри экземпляра.

```bash
# Получить интерактивную оболочку в контейнере
incus exec my-container -- /bin/sh

# Выполнить команду без входа в контейнер
incus exec my-container -- ls -la /tmp
```

### `incus stop` / `start` / `restart`

Управляет жизненным циклом экземпляра.

```bash
# Остановить экземпляр (graceful shutdown)
incus stop my-container

# Принудительно остановить
incus stop my-container --force

# Запустить
incus start my-container

# Перезапустить
incus restart my-container
```

### `incus delete`

Удаляет экземпляр. **Все данные будут утеряны**, если не используются отдельные тома.

```bash
# Удалить остановленный экземпляр
incus delete my-container

# Принудительно удалить запущенный
incus delete my-container --force
```

### `incus info`

Показывает подробную информацию об экземпляре.

```bash
incus info my-container
```

---

## ⚙️ Управление конфигурацией

### `incus config`

Позволяет просматривать и изменять конфигурацию экземпляров, профилей и сервера.

```bash
# Показать всю конфигурацию контейнера
incus config show my-container

# Показать расширенную конфигурацию (включая профили)
incus config show my-container --expanded

# Установить лимит памяти в 512MB
incus config set my-container limits.memory=512MB
```

### `incus profile`

Профили — это наборы конфигураций, которые можно применять к нескольким экземплярам.

```bash
# Показать список профилей
incus profile list

# Показать содержимое профиля 'default'
incus profile show default

# Создать новый профиль
incus profile create web-server

# Применить профиль к контейнеру
incus profile apply my-container web-server
```

---

## 📦 Управление образами

### `incus image`

Команды для работы с образами.

```bash
# Показать список локальных образов
incus image list

# Показать список образов в удаленном репозитории 'images:'
incus image list images:

# Скопировать образ из репозитория локально
incus image copy images:alpine/edge local: --alias my-alpine

# Удалить локальный образ
incus image delete my-alpine
```

---

## 💾 Управление хранилищем

### `incus storage`

Команды для управления пулами хранения и томами.

```bash
# Показать список пулов хранения
incus storage list

# Создать новый том 'my-data' в пуле 'default'
incus storage volume create default my-data

# Подключить том к контейнеру
incus storage volume attach default my-data my-container /data

# Показать список томов
incus storage volume list default
```

---

## 🌐 Управление сетями

### `incus network`

Команды для управления сетями.

```bash
# Показать список сетей
incus network list

# Создать новую bridge-сеть
incus network create my-net

# Подключить контейнер к сети
incus network attach my-net my-container
```
