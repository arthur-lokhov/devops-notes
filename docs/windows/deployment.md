---
title: Подготовка и автоматизация образов
hide:
  - toc
---

# 🔄 Подготовка и автоматизация образов

Основа современного управления Windows-инфраструктурой — это создание стандартизированных, готовых к развертыванию образов. Этот подход, известный как создание **"золотых образов"**, позволяет гарантировать, что каждая новая система будет настроена одинаково, безопасно и с нужным набором инструментов.

В этой главе мы рассмотрим ключевые инструменты и SRE-практики для автоматизации этого процесса.

---

## 🤖 Автоматическая установка: `autounattend.xml`

**Проблема, которую решает:** Ручная установка Windows — это медленный, подверженный ошибкам процесс. Каждая новая система может незначительно отличаться, что приводит к появлению **"серверов-снежинок"** (**snowflake servers**), которыми невозможно управлять централизованно.

**Решение:** Файлы ответов `autounattend.xml` — это фундамент для **полностью автоматической** (**unattended**) **установки Windows**. Они позволяют декларативно определить все параметры системы до начала установки.

**Когда использовать:**
- При развертывании на **"голое железо"** (**bare-metal**).
- Для создания базового образа виртуальной машины, который затем будет использоваться в `Packer`.

!!! example "Пример структуры `autounattend.xml`"

    Файл `autounattend.xml` содержит несколько этапов (passes). Ключевые из них:
    - `windowsPE`: Настройка диска и сетевых параметров до начала установки.
    - `specialize`: Настройка имени компьютера, домена и других уникальных параметров.
    - `oobeSystem`: Создание учетных записей, настройка языка и региона.

    ````xml
    <!-- Пример секции для создания локального администратора -->
    <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" ...>
      <UserAccounts>
        <LocalAccounts>
          <LocalAccount wcm:action="add">
            <Password>
              <Value>UABzAHMAdwBvAHIAZAA=</Value> <!-- (1) -->
              <PlainText>false</PlainText>
            </Password>
            <Group>Administrators</Group>
            <Name>localadmin</Name>
          </LocalAccount>
        </LocalAccounts>
      </UserAccounts>
    </component>
    ````

    1.  :key: Пароль должен быть представлен в формате **Base64** для безопасности.

---

## ✨ Создание "Золотых образов" с помощью Packer

**Проблема, которую решает:** Ручное создание образов не поддается версионированию, его сложно воспроизвести и легко допустить ошибку. Нужен автоматизированный, консистентный и версионируемый процесс.

**Решение:** **Packer** от HashiCorp — это индустриальный стандарт для реализации подхода **Infrastructure as Code (IaC)** при сборке образов. Он читает шаблон, автоматически создает ВМ, настраивает её и "запечатывает" в готовый образ.

### ⚙️ Конфигурация и Тестирование Образа (SRE Practice)

Профессиональный подход к сборке образов подразумевает, что процесс должен быть декларативным и тестируемым.

!!! abstract "Декларативная конфигурация: PowerShell DSC и Ansible"
    **Проблема:** Простые скрипты (`winget install...`, `Set-ItemProperty...`) являются **императивными** — они описывают, *как* что-то сделать. Если скрипт упадет на полпути, система останется в полунастроенном состоянии. 

    **Решение:** Использование **декларативных** инструментов, таких как **PowerShell DSC** или **Ansible**, через провижинеры `Packer`. Вы описываете, *каким* должен быть результат (**Desired State**), а инструмент сам определяет, как этого достичь. Это гарантирует **идемпотентность**: повторный запуск конфигурации не изменит систему, если она уже в нужном состоянии.

    !!! example "Пример: Установка Web-Server с помощью DSC в Packer"

        1.  **Конфигурация DSC (`WebServer.ps1`):**
            ```powershell
            Configuration WebServer
            {
                Import-DscResource -ModuleName 'PSDscResources'

                Node 'localhost'
                {
                    WindowsFeature 'IIS'
                    {
                        Ensure = 'Present',
                        Name   = 'Web-Server'
                    }

                    Firewall 'Allow-HTTP'
                    {
                        Ensure      = 'Present',
                        DisplayName = 'Allow HTTP',
                        Direction   = 'Inbound',
                        Protocol    = 'TCP',
                        LocalPort   = '80'
                    }
                }
            }
            ```

        2.  **Шаблон Packer (`build.pkr.hcl`):**
            ```hcl
            provisioner "dsc" {
              configuration_file = "./WebServer.ps1"
              configuration_name = "WebServer"
            }
            ```

!!! success "Тестирование и Quality Gates: Pester"
    **Проблема:** Как убедиться, что собранный образ соответствует требованиям безопасности и функциональности, *перед* его развертыванием?

    **Решение:** Использование фреймворка **Pester** для проведения автоматического тестирования внутри `Packer`. Это **критически важный Quality Gate**.

    **Когда использовать:** В пайплайне `Packer`, после шага конфигурации, но до `Sysprep`.

    !!! example "Пример теста на Pester (`Image.Tests.ps1`)"

        ```powershell
        # Тест проверяет, что IIS установлен и брандмауэр настроен
        Describe 'Web Server Image Validation' {
            It 'Should have IIS feature installed' {
                Get-WindowsFeature -Name 'Web-Server' | Should -HaveProperty 'Installed' -WithValue $true
            }

            It 'Should have an inbound firewall rule for HTTP' {
                Get-NetFirewallRule -DisplayName 'Allow HTTP' | Should -Not -BeNull
            }
        }
        ```

        В `Packer` этот тест запускается через `powershell` провижинер. Если тесты провалятся, сборка образа будет прервана.
        ```hcl
        provisioner "powershell" {
          script = "./run-tests.ps1" # Скрипт, который вызывает Invoke-Pester
        }
        ```

### 🏁 Завершение и Sysprep

**Проблема, которую решает:** Клонирование машин без их обобщения приводит к конфликтам в сети и домене из-за дублирования уникальных идентификаторов (**SID**).

**Решение:** **Sysprep (System Preparation Tool)** — обязательный последний шаг. Он удаляет **SID**, сбрасывает таймер активации и готовит систему к созданию образа. `Packer` вызывает `Sysprep` автоматически.

### 🚀 CI/CD и Управление Артефактами

**Проблема, которую решает:** Ручная сборка образов — это долго, не интегрировано в разработку и нет истории изменений.

**Решение:** Встраивание сборки образов в **CI/CD пайплайны** (`GitHub Actions`, `Azure DevOps`).

**Типичный рабочий процесс:**
1.  **Триггер:** Инженер вносит изменения в код конфигурации (`DSC`/`Ansible`) и отправляет их в `Git`.
2.  **Сборка:** CI/CD система автоматически запускает `packer build`.
3.  **Версионирование:** Образу присваивается версия, например, `v2025.09.25.1`.
4.  **Публикация:** Образ загружается в репозиторий артефактов (**Azure Compute Gallery**, **AWS AMI**), откуда он готов к развертыванию.

!!! example "Концептуальный пайплайн в GitHub Actions"

    ```yaml
    name: Build Windows Image

    on: 
      push:
        paths:
          - 'packer/**'

    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v3
        - name: Setup Packer
          uses: hashicorp/setup-packer@main

        - name: Build Image
          run: packer build -var="client_id=${{ secrets.AZURE_CLIENT_ID }}" template.pkr.hcl
    ```

---

## 🛠️ Обслуживание образов и драйверов (ADK)

Для низкоуровневого обслуживания образов Microsoft предоставляет **Windows Assessment and Deployment Kit (ADK)** — набор профессиональных инструментов.

### 💿 Offline-обслуживание с DISM

**Проблема, которую решает:** Как применить критическое обновление безопасности или добавить драйвер для нового оборудования в существующий образ, не разворачивая и не загружая его?

**Решение:** **DISM (Deployment Image Servicing and Management)** — утилита для **offline-обслуживания** образов (`.wim`, `.vhdx`).

!!! note "DISM (Offline) vs. PnPUtil (Online)"
    - **DISM** работает с **образами (offline)**. Идеален для подготовки образа *до* развертывания.
    - **PnPUtil** работает с **работающей системой (online)**. Идеален для скриптов, выполняемых *после* развертывания.

!!! example "Пример: Интеграция драйверов и обновлений"

    ```powershell
    # 1. Смонтировать образ
    Mount-WindowsImage -ImagePath "C:\images\install.wim" -Index 1 -Path "C:\mount"

    # 2. Добавить драйвер
    Add-WindowsDriver -Path "C:\mount" -Driver "C:\drivers" -Recurse

    # 3. Добавить пакет обновления
    Add-WindowsPackage -Path "C:\mount" -PackagePath "C:\updates\windows-update.msu"

    # 4. Сохранить изменения
    Dismount-WindowsImage -Path "C:\mount" -Save
    ```

### 🔌 Online-управление драйверами с PnPUtil

**Проблема, которую решает:** На уже развернутой системе нужно установить драйвер для нового устройства или провести аудит установленных сторонних драйверов.

**Решение:** **PnPUtil.exe** — утилита для управления драйверами в **работающей системе**.

**Когда использовать:**
- В скриптах `PowerShell DSC` или `Ansible` для динамической установки драйверов.
- Для аудита и траблшутинга (например, найти все неподписанные драйверы).

!!! example "Пример: Аудит и установка драйвера"

    ```powershell
    # Найти все сторонние драйверы в системе
    pnputil /enum-drivers | Select-String "Microsoft" -NotMatch

    # Установить новый драйвер из .inf файла
    pnputil /add-driver "C:\new_driver\driver.inf" /install
    ```
