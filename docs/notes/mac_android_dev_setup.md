---
title: Конфигурация macOS
---

# 📖 Конфигурация macOS

Это не просто очередное руководство.

Это история о том, как мы вместе пройдём путь от элегантной, но полной компромиссов **macOS**, до бескомпромиссно мощной, гибкой и защищённой среды для Android-разработчика.

---

## 💿 Под капотом macOS

Прежде чем мы начнём менять систему, важно понять её основы.

**macOS** — это не просто красивый интерфейс, а сложная экосистема с мощными механизмами защиты и встроенными утилитами.

### 🛡️ Внутренняя защита

Любая крепость начинается с дозорных на стенах.

В **macOS** их роль выполняют *[System Integrity Protection (SIP)](https://en.wikipedia.org/wiki/System_Integrity_Protection)* и *[Gatekeeper](https://support.apple.com/guide/security/gatekeeper-and-runtime-protection-sec5599b66df/web)*.

!!! question "Союзники, а не враги"

    **System Integrity Protection (SIP)** — это страж, который защищает ядро и самые важные системные файлы от любых изменений, даже от вас.

    **Gatekeeper** следит, чтобы вы случайно не запустили программное обеспечение от недоверенного разработчика.

    Пока эти двое на посту, мы можем смело экспериментировать в нашем "внутреннем дворе" — пользовательском пространстве (`/opt/homebrew`, `~/Applications`), зная, что сердце системы под надёжной защитой.

---

## 🛠️ Возведение крепости

Фундамент безопасности закладывается не только автоматизацией, но и несколькими критически важными ручными настройками.

Эти шаги требуют вашего прямого участия.

### 🔐 FileVault

Ваши данные — это главное сокровище.
Если ноутбук попадёт в чужие руки, единственное, что встанет между злоумышленником и вашими файлами — это шифрование.

!!! question "Почему важно включить?"

    **FileVault** — это не опция, а обязательный шаг.
    
    Его необходимо включить в **Системные настройки** -> **Конфиденциальность и безопасность**.
    
    Процесс активации генерирует уникальный ключ восстановления, который вы должны сохранить в надёжном месте.

### 🔥 Брандмауэр

Встроенный фаервол **macOS** отлично блокирует **входящие** соединения, но его нужно активировать и правильно настроить.

!!! question "Почему важно включить?"

    Активация фаервола — это базовое правило гигиены.
    
    Включите его в **Системные настройки** -> **Сеть** -> **Брандмауэр**.

    Важно также включить **"невидимый режим" (Stealth Mode)**.
    В этом режиме система не будет отвечать на запросы `ping` и другие попытки сканирования, делая ваш Mac "невидимым" в сети.

### 🛡️ Стек Objective-See

Встроенный брандмауэр — это лишь первая линия обороны, контролирующая **входящие** соединения.
Но что насчёт приложений, которые уже работают в системе?

Они могут без вашего ведома отправлять данные в интернет, прописываться в автозагрузку или следить за вами.

Здесь на помощь приходит бесплатный стек утилит с открытым исходным кодом от **Objective-See**, созданный ведущим экспертом по безопасности macOS, **Патриком Уордлом**.
Эти инструменты дают вам полный контроль над системой. Все они включены в наш `Brewfile` для автоматической установки, который вы найдете ниже.

!!! abstract "Почему это важно?"

    Современные угрозы редко пытаются "проломить" фаервол снаружи.
    Чаще они проникают под видом легитимных программ и действуют изнутри.
    
    Стек Objective-See — это ваша служба внутренней безопасности, которая отслеживает подозрительную активность на всех уровнях.

- **LuLu (Контроль сети):** Это фаервол для **исходящих** соединений.

    Как только любое приложение попытается отправить данные в сеть, `LuLu` покажет уведомление и спросит вашего разрешения.

    Вы решаете, кому можно доверять, а кому — нет.

- **BlockBlock (Контроль автозагрузки в реальном времени):** Это ваш часовой на страже "точек постоянства" (persistence locations).

    Если какая-либо программа попытается прописать себя в автозагрузку, чтобы запускаться при каждом старте системы, `BlockBlock` немедленно заблокирует эту попытку и предупредит вас.

- **KnockKnock (Аудит автозагрузки):** Если `BlockBlock` — это часовой, то `KnockKnock` — это аудитор.

    Он сканирует все известные места, где может прятаться автоматически запускаемое ПО, и показывает вам полный список.

    Это отличный способ проверить, не прописался ли кто-то в систему до установки `BlockBlock`.

- **ReiKey (Защита от кейлоггеров):** Эта утилита специфически нацелена на обнаружение перехватчиков клавиатуры (keyloggers).

    `ReiKey` сканирует систему на наличие известных техник перехвата и предупреждает, если находит что-то подозрительное.

- **Oversight (Контроль камеры и микрофона):** Простая, но критически важная утилита.

    `Oversight` постоянно следит за доступом к вашей веб-камере и микрофону.

    Как только какое-либо приложение активирует их, вы мгновенно получаете уведомление.

---

## 🍺 Brewfile

Крепость возведена. Теперь пора наполнить её арсеналом.

Вместо того чтобы устанавливать инструменты по одному, мы применим подход **"инфраструктура как код"** к нашему софту, используя **Homebrew** и `Brewfile`.

### 🚪 Установка Homebrew

**Homebrew** — это де-факто стандартный менеджер пакетов для **macOS**.

Он будет нашим единственным порталом для установки всего, от утилит командной строки до полноценных приложений.

!!! example "Установка Homebrew"

    ```bash
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```

### 📜 Создание Brewfile

`Brewfile` — это манифест, описывающий всё программное обеспечение, которое должно быть установлено в системе.

Это позволяет нам установить десятки инструментов одной командой.

??? example "Пример Brewfile для Android-разработчика"

    ```ruby
    #-------------------------------------------------------------------------------
    # 🍻 Homebrew Taps - Additional formula repositories.
    #-------------------------------------------------------------------------------
    tap "leoafarias/fvm"

    #-------------------------------------------------------------------------------
    # ⚙️ Core CLI Tools - Basic command-line utilities, mostly GNU versions.
    #-------------------------------------------------------------------------------
    brew "coreutils"    # GNU File, Shell, and Text utilities (provides gls, gcat, etc.).
    brew "findutils"    # Collection of GNU find, xargs, and locate.
    brew "gnu-getopt"   # GNU getopt utility for parsing command-line options.
    brew "gnu-indent"   # GNU indent utility to format C source code.
    brew "gnu-sed"      # GNU implementation of the sed stream editor.
    brew "gnu-tar"      # GNU tar archiving utility.
    brew "gnu-which"    # GNU implementation of the which utility.
    brew "gawk"         # GNU awk utility.
    brew "grep"         # GNU grep pattern matching utility.
    brew "stow"         # Symlink farm manager for managing dotfiles.
    brew "gpg"          # GNU Privacy Guard for encryption and signing.
    brew "pinentry-mac" # PIN entry dialog for GnuPG on macOS.
    brew "unar"         # Universal command-line unarchiver.
    brew "unzip"        # Utility for extracting and viewing files in ZIP archives.

    #-------------------------------------------------------------------------------
    # 🌐 Network Tools - Utilities for network diagnostics and data transfer.
    #-------------------------------------------------------------------------------
    brew "curl"         # Command-line tool for transferring data with URL syntax.
    brew "wget"         # Network utility to retrieve files from the Web.
    brew "xh"           # A friendly and fast tool for sending HTTP requests (curl alternative).
    brew "gnutls"       # GNU Transport Layer Security (TLS) Library.
    brew "iproute2mac"  # A macOS alternative to Linux's ip command (provides `ip a`).

    #-------------------------------------------------------------------------------
    # 🐚 Shell & Terminal - Tools to enhance the shell and terminal experience.
    #-------------------------------------------------------------------------------
    cask "iterm2"       # A replacement for Terminal and the successor to iTerm.
    brew "bash"         # Bourne-Again SHell, a UNIX command-line interpreter.
    brew "zsh"          # Z Shell, an extended Bourne shell with many improvements.
    brew "starship"     # The minimal, blazing-fast, and infinitely customizable prompt for any shell.
    brew "atuin"        # Magical shell history, synced, searchable, and structured.
    brew "zoxide"       # A smarter cd command that learns your habits.
    brew "fzf"          # A command-line fuzzy finder.

    #-------------------------------------------------------------------------------
    # 🔄 Modern CLI Alternatives - Upgraded versions of common command-line tools.
    #-------------------------------------------------------------------------------
    brew "eza"          # A modern replacement for ls.
    brew "bat"          # A cat(1) clone with wings (syntax highlighting and Git integration).
    brew "fd"           # A simple, fast and user-friendly alternative to find.
    brew "ripgrep"      # A line-oriented search tool that recursively searches for a regex pattern.
    brew "tldr"         # Collaborative cheatsheets for console commands (a simpler man).
    brew "htop"         # Interactive process viewer (classic).
    brew "btop"         # A modern and feature-rich resource monitor (htop replacement).
    brew "fastfetch"    # A fast, feature-rich, and customizable system information tool.
    brew "dust"         # A more intuitive version of du.
    brew "jq"           # Command-line JSON processor.
    brew "yq"           # Command-line YAML, JSON, and XML processor (jq's sibling).

    #-------------------------------------------------------------------------------
    # 🧑‍💻 Development & Git - Tools for coding and version control.
    #-------------------------------------------------------------------------------
    brew "mise"         # Polyglot tool version manager (replaces asdf, nvm, etc.).
    brew "fvm"          # Flutter Version Management.
    brew "xcodes"       # A tool to install and switch between multiple versions of Xcode.
    brew "git"          # Distributed version control system.
    brew "jj"           # A Git-compatible DVCS that is both simple and powerful.
    brew "gh"           # Official GitHub CLI tool.
    brew "delta"        # A syntax-highlighting pager for git, diff, and grep output.
    brew "lua"          # Lightweight, high-level, multi-paradigm programming language.

    #-------------------------------------------------------------------------------
    # 🏗️ Infrastructure & Containers - DevOps, IaaC, and containerization tools.
    #-------------------------------------------------------------------------------
    brew "colima"           # Container runtimes on macOS with minimal setup.
    brew "docker"           # Platform for developing, shipping, and running applications in containers.
    brew "docker-compose"   # Tool for defining and running multi-container Docker applications.
    brew "docker-buildx"    # Docker CLI plugin for extended build capabilities with BuildKit.

    #-------------------------------------------------------------------------------
    # 🤖 AI Tools - Command-line tools powered by AI.
    #-------------------------------------------------------------------------------
    brew "gemini-cli"   # Google Gemini CLI.

    #-------------------------------------------------------------------------------
    # 🖥️ GUI Applications - All the graphical user interface apps.
    #-------------------------------------------------------------------------------
    # Window Management
    # Raycast is installed manually
    # iBar is installed from App Store
    cask "alt-tab"          # Windows-like alt-tab behavior.
    cask "topnotch"         # Hides the notch on new MacBooks.
    cask "stats"            # System monitor in your menu bar.
    cask "mos"              # Smooth scrolling and mouse direction control.

    # Browsers & Communication
    cask "vivaldi"          # A feature-rich, customizable web browser.
    cask "discord"          # All-in-one voice and text chat for gamers.
    cask "telegram-desktop" # Secure and fast messaging app.

    # Productivity & Utilities
    cask "bitwarden"        # Password manager.
    cask "pearcleaner"      # Utility to clean and uninstall applications.
    cask "onlyoffice"       # Office suite.
    cask "the-unarchiver"   # The best unarchiving tool for macOS.
    cask "obs"              # Free and open source software for video recording and live streaming.
    cask "shottr"           # Advanced screenshot tool with annotation features.
    cask "xcodes-app"       # Xcode version manager GUI.
    cask "iina"             # Modern media player for macOS.
    cask "balenaetcher"     # Flash OS images to SD cards & USB drives.

    # Virtualization
    cask "utm"              # Virtual machine host for macOS.
    cask "genymotion"       # Android emulator for app development and testing.

    #-------------------------------------------------------------------------------
    # 🛡️ Security Tools - Utilities for system security and monitoring.
    #-------------------------------------------------------------------------------
    cask "lulu"             # Free macOS firewall to block unknown outgoing connections.
    cask "blockblock"       # Monitors common persistence locations and alerts on additions.
    cask "knockknock"       # Displays persistently installed software to uncover malware.
    cask "reikey"           # Scans for and alerts on keyboard event taps (keyloggers).
    cask "oversight"        # Monitors the mic and webcam to alert on activity.  

    #-------------------------------------------------------------------------------
    # 🔡 Fonts - Custom fonts for the terminal and editors.
    #-------------------------------------------------------------------------------
    cask "font-departure-mono-nerd-font" # Patched font with icons for development.

    #-------------------------------------------------------------------------------
    # 📦 SDKs - Software Development Kits.
    #-------------------------------------------------------------------------------
    ```

### 🚀 Установка

После создания `Brewfile`, вы можете установить всё одной командой:

```bash
brew bundle install
```

---

## 💻 Настройка терминала

Наш `Brewfile` установил целый арсенал мощных утилит командной строки.
Но чтобы превратить стандартный терминал в настоящий командный центр, их нужно активировать и настроить.

Эта конфигурация для `zsh` (который мы установили через `Brewfile`) превращает терминал из простого исполнителя команд в умного помощника.

Она добавляет автодополнение, подсветку синтаксиса, удобные псевдонимы (алиасы) для часто используемых команд и интегрирует современные инструменты, такие как `eza` (замена `ls`), `bat` (замена `cat`) и `zoxide` (умный `cd`).

Создайте или откройте файл `~/.zshrc` и добавьте в него следующую конфигурацию.

??? example "Пример .zshrc для мощного терминала"

    ```bash
    # ==============================================================================
    # 📜 Scripts
    # ==============================================================================
    # Source custom scripts. This is done first to make helper functions available.
    # ------------------------------------------------------------------------------

    # Use a robust path relative to the home directory.
    if [ -f "$HOME/dotfiles/scripts/console-colours.sh" ]; then
      source "$HOME/dotfiles/scripts/console-colours.sh"
    fi

    if [ -f "$HOME/dotfiles/scripts/linuxify.sh" ]; then
      source "$HOME/dotfiles/scripts/linuxify.sh"
    fi

    # ==============================================================================
    # 📦 Package Managers & Dependencies
    # ==============================================================================
    # Installation checks for essential tools to ensure the environment is ready.
    # ------------------------------------------------------------------------------

    ## Homebrew: Install if not present
    if ! command -v brew &> /dev/null; then
        show_info "Installing Homebrew..."
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
        show_success "Installing Homebrew...Done!"
    fi

    # ==============================================================================
    # ⚙️ Environment Variables
    # ==============================================================================
    # Configure the shell environment.
    # ------------------------------------------------------------------------------

    ## Homebrew: Set path and cache prefix for performance.
    BREW_HOME=$(brew --prefix)
    export PATH="${BREW_HOME}/bin:$PATH"
    export MANPATH="${BREW_HOME}/share/man:$MANPATH"
    export INFOPATH="${BREW_HOME}/share/info:$INFOPATH"

    ## General Exports
    export GOOGLE_CLOUD_PROJECT="viderbe"

    # Java exports
    export SDKMAN_DIR="$HOME/.sdkman"

    # Android exports
    export ANDROID_HOME=$HOME/Library/Android/sdk
    export PATH=$PATH:$ANDROID_HOME/emulator
    export PATH=$PATH:$ANDROID_HOME/platform-tools
    export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin

    ## Zap Plugin Manager: Define config file location.
    ZAP_CONFIG_FILE="${XDG_DATA_HOME:-$HOME/.local/share}/zap/zap.zsh"

    # Starship: Define config file location
    export STARSHIP_CONFIG="$HOME/.config/starship/starship.toml"

    # ==============================================================================
    # 🔐 GPG & SSH
    # ==============================================================================
    # GPG & SSH agent configuration.
    # ------------------------------------------------------------------------------

    export GPG_TTY=$(tty)
    gpg-connect-agent updatestartuptty /bye >/dev/null 2>&1 >/dev/null
    unset SSH_AGENT_PID
    export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)

    # ==============================================================================
    # 🐚 Shell Behavior (setopt)
    # ==============================================================================
    # Fine-tune Zsh behavior for a better interactive experience.
    # ------------------------------------------------------------------------------

    setopt AUTO_CD                # cd into a directory just by typing its name.
    setopt AUTO_PUSHD             # Automatically add directories to the stack.
    setopt PUSHD_IGNORE_DUPS      # Don't add duplicate directories to the stack.
    setopt EXTENDED_GLOB          # Enable more powerful glob patterns.
    setopt NUMERIC_GLOB_SORT      # Sort filenames numerically (e.g., file1, file2, file10).
    setopt HIST_IGNORE_ALL_DUPS   # Don't save duplicate commands in history.
    setopt HIST_REDUCE_BLANKS     # Remove superfluous blanks from history entries.

    # ==============================================================================
    # 💡 Aliases
    # ==============================================================================
    # Define shell aliases for quicker commands.
    # ------------------------------------------------------------------------------

    ### General Convenience
    alias ..="cd .."
    alias ...="cd ../.."
    alias ports="lsof -i -P -n | grep LISTEN" # Show open ports
    alias ip="ip -c" # Colorized output for iproute2mac
    alias update="brew bundle install --file=~/Brewfile && brew bundle cleanup" # Update all apps from Brewfile

    ### Alts: Modern alternatives to common commands.
    alias cd="z" # Zoxide
    alias l="eza -l --git -a --icons"
    alias lt="eza --tree --level=2 --long --git --icons"
    alias ltree="eza --tree --level=2 --git --icons"
    alias cat="bat"
    alias du="dust"

    # ==============================================================================
    # Functions
    # ==============================================================================
    # Define custom functions for more complex tasks.
    # ------------------------------------------------------------------------------

    # Create a directory and immediately cd into it.
    mkcd() {
      mkdir -p "$1" && cd "$1"
    }

    # Quickly create a backup of a file.
    backup() {
      cp "$1" "$1_$(date +%Y-%m-%d).bak"
    }

    # ==============================================================================
    # 🔌 Zsh Plugin Manager (Zap)
    # ==============================================================================
    # Zsh plugin management.
    # ------------------------------------------------------------------------------

    ## Auto-install Zap if not present
    if [ ! -f "$ZAP_CONFIG_FILE" ]; then
      show_info "Installing Zap Plugin Manager for Zsh..."
      zsh <(curl -sL https://raw.githubusercontent.com/zap-zsh/zap/master/install.zsh) --keep
      show_success "Installing Zap Plugin Manager for Zsh...Done!"
    fi

    ## Load Zap
    [ -f "$ZAP_CONFIG_FILE" ] && source "$ZAP_CONFIG_FILE"

    ## Plugins
    plug "zsh-users/zsh-autosuggestions"
    plug "zsh-users/zsh-syntax-highlighting"
    plug "zsh-users/zsh-completions"

    # ==============================================================================
    # 🤖 Completions
    # ==============================================================================
    # Configure Zsh completions.
    # ------------------------------------------------------------------------------

    ## Initialize the completion system
    autoload -U compinit
    compinit

    ## Source tool-specific completions
    if command -v docker &> /dev/null; then
      source <(docker completion zsh)
    fi

    [[ -s "$HOME/.sdkman/bin/sdkman-init.sh" ]] && source "$HOME/.sdkman/bin/sdkman-init.sh"

    # ==============================================================================
    # 🚀 Evals & Activations
    # ==============================================================================
    # Final activations and evaluations. This should be at the end of the file.
    # ------------------------------------------------------------------------------

    # zoxide: A smarter cd command
    eval "$(zoxide init zsh)"

    # atuin: Magical shell history
    eval "$(atuin init zsh)"

    # starship: Cross-shell prompt
    eval "$(starship init zsh)"
    ```

### ✨ Starship

В файле `.zshrc` мы указали путь к конфигу Starship: `export STARSHIP_CONFIG="$HOME/.config/starship/starship.toml"`.

Starship — это минималистичная, быстрая и бесконечно настраиваемая командная строка для любой оболочки.

Приведенная ниже конфигурация создает двухстрочную строку приглашения, которая является чистой, информативной и показывает релевантный контекст, такой как каталог, ветка git, версия python/go и т.д.

??? example "$HOME/.config/starship/starship.toml"

    ```toml
    "$schema" = 'https://starship.rs/config-schema.json'

    palette = "zenbones"

    [palettes.zenbones]
    red = "#DE6E7C"
    yellow = "#B77E64"
    green = "#819B69"
    blue = "#6099C0"
    pink = "#B279A7"
    cyan = "#66A5AD"
    text = "#BBBBBB"
    subtext = "#3D3839"
    bg = "#191919"

    # Формат: ровно один пробел между модулями
    format = "$directory $git_branch $git_status $all$line_break$character"

    [directory]
    style = "bold blue"
    format = "in [$path]($style) [$read_only]($style)"
    truncation_length = 4
    truncation_symbol = "…/"
    read_only = "ro"

    [git_branch]
    symbol = "git:"
    style = "bold yellow"
    # format = "on [$symbol$branch]($style) "
    format = "on [$symbol$branch]($style)"

    [git_status]
    style = "yellow"
    format = " [$all_status$ahead_behind]($style) "
    ahead = ">"
    behind = "<"
    diverged = "<>"
    stashed = "*"
    modified = "!"
    staged = "+"
    renamed = "r"
    deleted = "x"

    [kubernetes]
    symbol = "k8s:"
    style = "bold red"
    format = "via [$symbol$context(::$namespace)]($style) "
    disabled = false

    [docker_context]
    symbol = "docker:"
    style = "bold blue"
    format = "via [$symbol$context]($style) "

    [terraform]
    symbol = "tf:"
    style = "bold pink"
    format = "with [$symbol$workspace]($style) "

    [aws]
    symbol = "aws:"
    style = "bold yellow"
    format = "on [$symbol$profile]($style) "

    [golang]
    symbol = "go:"
    style = "bold blue"
    format = "using [$symbol($version)]($style) "

    [python]
    symbol = "py:"
    style = "bold yellow"
    format = "using [$symbol($version)]($style) "

    [lua]
    symbol = "lua:"
    style = "bold green"
    format = "using [$symbol($version)]($style) "

    [character]
    success_symbol = "[>](bold green) "
    error_symbol = "[x](bold red) "
    vimcmd_symbol = "[<](bold green) "
    ```

### 📜 Скрипты

Для лучшей организации и чистоты основного конфигурационного файла (`.zshrc`), часть логики вынесена во вспомогательные скрипты.
Они подключаются в самом начале `.zshrc`.

??? example "$HOME/scripts/console-colours.sh"

    Этот скрипт определяет переменные для цветного вывода в терминале и функции-обёртки для отображения сообщений разного уровня (ошибка, предупреждение, успех).
    
    Это делает вывод других скриптов более наглядным и читаемым.

    ```bash
    #!/usr/bin/env bash

    ERROR=`tput setaf 1`
    WARN=`tput setaf 3`
    INFO=`tput setaf 4`
    SUCCESS=`tput setaf 2`
    BOLD=`tput smso`
    UNBOLD=`tput rmso`
    ENDMARKER=`tput sgr0`

    show_error() {
        echo -e "${BOLD}${ERROR}ERROR:${UNBOLD} ${ERROR}$1 ${ENDMARKER}" 1>&2
    }

    show_warning() {
        echo -e "${BOLD}${WARN}WARNING:${UNBOLD} ${WARN}$1 ${ENDMARKER}"
    }

    show_info() {
        echo -e "${INFO}$1 ${ENDMARKER}"
    }

    show_success() {
        echo -e "${SUCCESS}$1 ${ENDMARKER}"
    }

    die() {
         show_error "$1" 1>&2
         exit 1
    }
    ```

??? example "$HOME/scripts/linuxify.sh"

    macOS поставляется с устаревшими версиями многих стандартных утилит BSD, в то время как в мире Linux используются утилиты GNU с более богатым функционалом.
    
    Homebrew устанавливает GNU-версии с префиксом `g` (например, `gls`, `gcat`).

    Этот скрипт создает псевдонимы (aliases), чтобы при вызове стандартных команд (`ls`, `cat`) на самом деле использовались их мощные GNU-аналоги.
    Это создаёт привычную и более функциональную среду для тех, кто работает с Linux.

    ```bash
    #!/usr/bin/env zsh

    # ==============================================================================
    # 🐧 Linuxify Aliases
    # ==============================================================================
    # This file contains a comprehensive list of aliases to map standard command
    # names to their GNU versions, which are typically prefixed with 'g' by Homebrew.
    # This helps create a more consistent, Linux-like environment on macOS.
    # ------------------------------------------------------------------------------

    # Coreutils - Comprehensive list of core GNU utilities
    alias awk='gawk'
    alias b2sum='gb2sum'
    alias base32='gbase32'
    alias base64='gbase64'
    alias basename='gbasename'
    alias cat='gcat'
    alias chcon='gchcon'
    alias chgrp='gchgrp'
    alias chmod='gchmod'
    alias chown='gchown'
    alias chroot='gchroot'
    alias cksum='gcksum'
    alias comm='gcomm'
    alias cp='gcp -v'
    alias csplit='gcsplit'
    alias cut='gcut'
    alias date='gdate'
    alias dd='gdd'
    alias df='gdf -h'
    alias dir='gdir'
    alias dircolors='gdircolors'
    alias dirname='gdirname'
    alias du='gdu -h'
    # alias echo='gecho'      # Disabled: Conflicts with shell builtin, can break scripts.
    alias env='genv'
    alias expand='gexpand'
    alias expr='gexpr'
    alias factor='gfactor'
    alias false='gfalse'
    alias fmt='gfmt'
    alias fold='gfold'
    alias groups='ggroups'
    alias head='ghead'
    alias hostid='ghostid'
    alias id='gid'
    alias install='ginstall'
    alias join='gjoin'
    alias kill='gkill'
    alias link='glink'
    alias ln='gln'
    alias logname='glogname'
    alias ls='gls --color=auto'
    alias md5sum='gmd5sum'
    alias mkdir='gmkdir'
    alias mkfifo='gmkfifo'
    alias mknod='gmknod'
    alias mktemp='gmktemp'
    alias mv='gmv -v'
    alias nice='gnice'
    alias nl='gnl'
    alias nohup='gnohup'
    alias nproc='gnproc'
    alias numfmt='gnumfmt'
    alias paste='gpaste'
    alias pathchk='gpathchk'
    alias pinky='gpinky'
    alias pr='gpr'
    alias printenv='gprintenv'
    # alias printf='gprintf' # Disabled: Causes warnings with other shell tools like starship/atuin.
    alias ptx='gptx'
    alias pwd='gpwd'
    alias readlink='greadlink'
    alias realpath='grealpath'
    alias rm='grm -v'
    alias rmdir='grmdir'
    alias runcon='gruncon'
    alias seq='gseq'
    alias sha1sum='gsha1sum'
    alias sha224sum='gsha224sum'
    alias sha256sum='gsha256sum'
    alias sha384sum='gsha384sum'
    alias sha512sum='gsha512sum'
    alias shred='gshred'
    alias shuf='gshuf'
    alias sleep='gsleep'
    alias sort='gsort'
    alias split='gsplit'
    alias stat='gstat'
    alias stdbuf='gstdbuf'
    alias stty='gstty'
    alias sum='gsum'
    alias sync='gsync'
    alias tac='gtac'
    alias tail='gtail'
    alias tee='gtee'
    alias test='gtest'
    alias timeout='gtimeout'
    alias touch='gtouch'
    alias tr='gtr'
    alias true='gtrue'
    alias truncate='gtruncate'
    alias tsort='gtsort'
    alias tty='gtty'
    alias uname='guname'
    alias unexpand='gunexpand'
    alias uniq='guniq'
    alias unlink='gunlink'
    alias uptime='guptime'
    alias users='gusers'
    alias vdir='gvdir'
    alias wc='gwc'
    alias who='gwho'
    alias whoami='gwhoami'
    alias yes='gyes'

    # Findutils
    alias find='gfind'
    alias locate='glocate'
    alias updatedb='gupdatedb'
    alias xargs='gxargs'

    # Grep
    alias egrep='gegrep'
    alias fgrep='gfgrep'
    alias grep='ggrep --color=auto'

    # Sed
    alias sed='gsed'

    # Tar
    alias tar='gtar'

    # Which
    alias which='gwhich'

    # Indent
    alias indent='gindent'

    # Getopt
    alias getopt='ggetopt'
    ```

---

## ⚙️ `defaults` в MacOs

**macOS** скрывает множество настроек, которые недоступны через стандартный интерфейс "Системных настроек".

Команда `defaults` в терминале — это ключ к этим скрытым возможностям.
Она позволяет напрямую изменять файлы `.plist`, в которых хранятся настройки системы и приложений.

Это мощный инструмент для кастомизации, который позволяет ускорить анимации, улучшить поведение Finder, настроить Dock и многое другое.

!!! warning "Действуйте с осторожностью"

    Изменение настроек через `defaults` может привести к неожиданному поведению, если вы не знаете, что делаете.

    Всегда полезно делать резервные копии, прежде чем вносить серьезные изменения.

Вручную вводить десятки команд неудобно.

Сообщество создало готовые скрипты, которые применяют сотни полезных твиков.

Один из самых известных и полных — это [скрипт `.macos` от Mathias Bynens](https://github.com/mathiasbynens/dotfiles/blob/main/.macos).

Вы можете изучить его, выбрать то, что вам нужно, и создать свой собственный скрипт для автоматической настройки новой системы.

---

## ☕ Настройка Java

Для Android-разработки требуется **JDK (Java Development Kit)**.

Вместо того чтобы использовать Homebrew, мы воспользуемся `sdkman` — лучшим инструментом для управления несколькими версиями Java.

### 📥 Установка SDKMAN

`sdkman` устанавливается одной командой в терминале:

```bash
curl -s "https://get.sdkman.io" | bash
```

После установки следуйте инструкциям и откройте новое окно терминала.
`sdkman` сам пропишет себя в ваш `.zshrc`.

### ☕ Установка Java

Теперь вы можете легко установить любую нужную версию **JDK**.

Например, для современных Android-проектов рекомендуется **JDK 17**.

```bash
# Установить JDK 17 и сделать его версией по умолчанию
sdk install java 17.0.11-tem
sdk default java 17.0.11-tem
```

`sdkman` автоматически настроит все необходимые переменные окружения, включая `JAVA_HOME`.

---

## 🤖 Настройка Android SDK

Это самый важный шаг. Наша цель — сделать так, чтобы и **Android Studio**, и командная строка использовали **один и тот же Android SDK**. Это избавит от путаницы и дублирования.

### ❌ Что не нужно делать

Не устанавливайте `android-commandlinetools` отдельно через Homebrew. Это создаст второй, конфликтующий SDK. Мы будем использовать тот, что поставляется с **Android Studio**.

### ✅ Правильный путь

**Шаг 1: Первый запуск Android Studio**

После установки через `brew cask install android-studio`, запустите студию. Она предложит скачать и установить компоненты SDK. Согласитесь и позвольте ей завершить стандартную установку. По умолчанию SDK будет расположен в `~/Library/Android/sdk`.

**Шаг 2: Установка Command-Line Tools через Android Studio**

Теперь нам нужно, чтобы в этом SDK появились утилиты командной строки (`sdkmanager`, `avdmanager`).

1.  Откройте Android Studio.
2.  Перейдите в **Settings** (Настройки).
3.  Найдите раздел **Languages & Frameworks → Android SDK**.
4.  Переключитесь на вкладку **SDK Tools**.
5.  Поставьте галочку напротив **Android SDK Command-line Tools (latest)**.
6.  Нажмите **Apply**. Студия скачает и установит их в свой SDK.

**Шаг 3: Настройка `.zshrc`**

Это финальный шаг, который свяжет всё воедино. Откройте ваш файл `.zshrc` и добавьте пути к SDK, который использует Android Studio.

!!! example "Настройка переменных окружения в .zshrc"
    ```bash
    # Указываем Android Studio, где находится её SDK
    export ANDROID_HOME=$HOME/Library/Android/sdk

    # Добавляем инструменты SDK в PATH для доступа из консоли
    export PATH=$PATH:$ANDROID_HOME/emulator
    export PATH=$PATH:$ANDROID_HOME/platform-tools
    export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
    ```

**Результат:** Теперь, когда вы вызовете `sdkmanager` или `adb` в терминале, это будут те же самые утилиты, которые использует Android Studio. Обновления, сделанные в студии, будут доступны в консоли, и наоборот.

---

## ✨ Эпилог: Готово!

Поздравляю! Теперь у вас есть чистая, автоматизированная и сфокусированная среда для Android-разработки.

Вы объединили мощь **Homebrew** для установки приложений, `sdkman` для гибкого управления **Java** и добились идеальной синхронизации **Android SDK** между **Android Studio** и командной строкой.

Это надёжный фундамент, который сэкономит вам массу времени и нервов в будущем.
