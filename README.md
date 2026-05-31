# Zellij — конфигурация

Персональная конфигурация [Zellij](https://zellij.dev) для macOS.
Хранится в `~/.config/zellij/`.

```
~/.config/zellij/
├── config.kdl          # основной конфиг: кейбинды, тема, плагины, поведение
├── layouts/            # шаблоны вкладок и панелей
│   ├── dev.kdl
│   ├── k8s.kdl
│   └── ops.kdl
├── plugins/            # сторонние плагины (.wasm)
│   ├── zjstatus.wasm
│   ├── zellij-sessionizer.wasm
│   ├── vim-zellij-navigator.wasm
│   ├── zellij-autolock.wasm
│   └── harpoon.wasm
└── README.md
```

---

## Содержание

1. [Что такое Zellij](#что-такое-zellij)
2. [Установка](#установка)
3. [Быстрый старт](#быстрый-старт)
4. [Основные опции конфига](#основные-опции-конфига)
5. [Режимы (modes)](#режимы-modes)
6. [Кейбинды](#кейбинды)
7. [Сессии](#сессии)
8. [Layouts](#layouts)
9. [Плагины](#плагины) · [Дефолтный dev-набор](#дефолтный-dev-набор)
10. [Тема](#тема)
11. [Полезные команды CLI](#полезные-команды-cli)
12. [Решение проблем](#решение-проблем)

---

## Что такое Zellij

Zellij — мультиплексор терминала (как `tmux` или `screen`), который:

- Делит окно терминала на **панели** (panes) и группирует их во **вкладки** (tabs).
- Сохраняет работающие процессы в **сессиях**, к которым можно отключаться/переподключаться (`detach`/`attach`) — переживает закрытие терминала.
- Управляется через **модальные режимы** (как в Vim) — внизу всегда подсказка по текущему режиму.
- Поддерживает **layouts** — декларативные шаблоны рабочих окружений (KDL-файлы).
- Расширяется **плагинами** (WASM).

---

## Установка

```bash
brew install zellij        # macOS, Homebrew
zellij --version
```

Текущая версия конфигурации проверена на `zellij 0.44.3`.

Сделать стартом по умолчанию для нового окна терминала — добавить в `~/.zshrc`:

```bash
# Автозапуск zellij в интерактивных сессиях (кроме SSH/tmux/VSCode-терминала)
if [[ -z "$ZELLIJ" && -z "$TMUX" && -z "$VSCODE_INJECTION" && $- == *i* ]]; then
    zellij attach -c main || zellij
fi
```

---

## Быстрый старт

```bash
zellij                         # новая сессия со случайным именем
zellij --layout dev            # старт с layout dev.kdl
zellij -s work                 # сессия с именем work
zellij a work                  # attach к существующей сессии
zellij ls                      # список сессий
zellij kill-session work       # убить сессию
zellij delete-all-sessions     # удалить все сериализованные сессии
```

Внутри сессии:

| Действие | Бинд |
|---|---|
| Detach (отсоединиться, оставив сессию в фоне) | `Ctrl+o`, затем `d` |
| Quit (закрыть текущего клиента и завершить сессию) | `Ctrl+q` |
| Заблокировать ввод (отключить все бинды zellij) | `Ctrl+g` |
| Разблокировать | `Ctrl+g` |

---

## Основные опции конфига

Все настройки в `config.kdl`. Ключевое из текущей конфигурации:

| Опция | Значение | Что делает |
|---|---|---|
| `theme` | `gruvbox-dark` | Встроенная тёплая ретро-палитра |
| `session_serialization` | `true` | Структура сессии (вкладки, панели, cwd, команды) сохраняется на диск |
| `serialize_pane_viewport` | `true` | Дополнительно сохраняется видимое содержимое панелей |
| `copy_command` | `pbcopy` | Выделение через мышь летит прямо в системный буфер macOS |
| `show_startup_tips` | `false` | Не показывать подсказки при запуске |
| `default_mode` | `normal` (по умолчанию) | Стартуем в обычном режиме — все Alt-бинды активны |

Полезные опции, которые не включены, но могут пригодиться (раскомментировать в `config.kdl`):

```kdl
default_layout "dev"           // открывать сразу с layout dev
default_shell "/opt/homebrew/bin/fish"
pane_frames false              // убрать рамки вокруг панелей
mouse_mode false               // выключить мышь, если мешает выделению
simplified_ui true             // без спец. шрифтов (если иконки не рендерятся)
```

После правки `config.kdl` проверь синтаксис:

```bash
zellij setup --check
```

---

## Режимы (modes)

Zellij модальный. В каждом режиме свой набор биндов. Переход — `Ctrl+<буква>`.
В нижней панели всегда показано, где ты находишься.

| Префикс | Режим | Для чего |
|---|---|---|
| `Ctrl+p` | **pane** | Создание/закрытие/перемещение фокуса между панелями |
| `Ctrl+t` | **tab** | Управление вкладками |
| `Ctrl+n` | **resize** | Изменение размеров активной панели |
| `Ctrl+h` | **move** | Перетаскивание панели по сетке |
| `Ctrl+s` | **scroll** | Прокрутка буфера, переход в search |
| `Ctrl+o` | **session** | Сессии, плагины (session-manager, configuration и т.п.) |
| `Ctrl+b` | **tmux** | Эмуляция tmux-биндов (если привычно с tmux) |
| `Ctrl+g` | **locked** | Полная блокировка биндов — терминал получает все клавиши |
| `Ctrl+q` | — | Выход (Quit) |

Из любого режима: `Enter` или `Esc` → возврат в `normal`.

---

## Кейбинды

### Глобальные (работают везде кроме `locked`)

| Бинд | Действие |
|---|---|
| `Alt+←/→` или `Alt+h/l` | Фокус влево/вправо (между панелями и вкладками) |
| `Alt+↑/↓` или `Alt+k/j` | Фокус вверх/вниз между панелями |
| `Alt+n` | Новая панель (направление выбирается автоматически) |
| `Alt+t` | **Новая вкладка** *(кастом)* |
| `Alt+w` | **Закрыть текущую панель** *(кастом)* |
| `Alt+v` | **Split вправо** *(кастом)* |
| `Alt+s` | **Split вниз** *(кастом)* |
| `Ctrl+h/j/k/l` | **Бесшовная навигация с vim/nvim** *(vim-navigator)* |
| `Alt+g` | **Sessionizer** — пикер сессий по папкам |
| `Ctrl+y` | **Harpoon** — список «закладок» панелей |
| `Alt+z` | **Toggle autolock** (вручную в/из locked) |
| `Alt+m` | Войти в move-mode (раньше был `Ctrl+h`) |
| `Alt+f` | Toggle floating panes |
| `Alt+i` / `Alt+o` | Передвинуть вкладку влево/вправо |
| `Alt+[` / `Alt+]` | Предыдущий/следующий swap-layout |
| `Alt + +` / `Alt + -` | Увеличить/уменьшить размер активной панели |
| `Alt+p` | Добавить/убрать панель в группу |
| `Alt+Shift+p` | Toggle group marking |
| `Ctrl+g` | Заблокировать/разблокировать ввод |
| `Ctrl+q` | Quit |

### Pane mode (`Ctrl+p`, потом)

| Бинд | Действие |
|---|---|
| `h/j/k/l` или стрелки | Двигать фокус |
| `n` | Новая панель |
| `d` | Новая панель снизу |
| `r` | Новая панель справа |
| `s` | Новая стэк-панель |
| `x` | Закрыть фокусную панель |
| `f` | Полный экран для текущей панели |
| `w` | Toggle floating panes |
| `e` | Toggle embed / floating |
| `i` | Закрепить (pinned) floating-панель |
| `z` | Toggle pane frames |
| `c` | Переименовать панель |
| `p` | Переключить фокус (по кругу) |

### Tab mode (`Ctrl+t`, потом)

| Бинд | Действие |
|---|---|
| `n` | Новая вкладка |
| `x` | Закрыть вкладку |
| `r` | Переименовать вкладку |
| `1`…`9` | Перейти на вкладку N |
| `h/j/k/l` или стрелки | Между вкладками |
| `[` / `]` | Вынести панель во вкладку слева/справа |
| `b` | Вынести панель в новую вкладку |
| `s` | Toggle Sync (один ввод во все панели вкладки) |
| `Tab` | Переключиться на предыдущую активную вкладку |

### Resize mode (`Ctrl+n`, потом)

| Бинд | Действие |
|---|---|
| `h/j/k/l` или стрелки | Увеличить в указанную сторону |
| `H/J/K/L` | Уменьшить в указанную сторону |
| `+` / `-` / `=` | Универсальное увеличение/уменьшение |

### Move mode (`Ctrl+h`, потом)

| Бинд | Действие |
|---|---|
| `h/j/k/l` или стрелки | Двигать панель |
| `n` или `Tab` | Move pane (по умолчанию направление) |
| `p` | Move pane назад по порядку |

### Scroll mode (`Ctrl+s`, потом)

| Бинд | Действие |
|---|---|
| `↑/↓` или `k/j` | Прокрутка на строку |
| `PageUp` / `PageDown` или `Ctrl+b` / `Ctrl+f` | Постранично |
| `u` / `d` | Полстраницы вверх/вниз |
| `Ctrl+c` | В конец буфера и выйти |
| `s` | Перейти в search-режим |
| `e` | Открыть scrollback в `$EDITOR` (правится в `scrollback_editor`) |

### Search mode (внутри scroll → `s`)

| Бинд | Действие |
|---|---|
| Ввод текста | Поиск |
| `n` / `p` | Следующее / предыдущее совпадение |
| `c` | Toggle case-sensitivity |
| `o` | Toggle whole word |
| `w` | Toggle wrap |

### Session mode (`Ctrl+o`, потом)

| Бинд | Действие |
|---|---|
| `d` | Detach текущего клиента |
| `w` | Открыть **session-manager** (список и переключение сессий) |
| `c` | Открыть **configuration** (настройки в TUI) |
| `p` | Открыть **plugin-manager** |
| `l` | Открыть **layout-manager** |
| `a` | About |
| `s` | Поделиться сессией (zellij share) |

### Tmux mode (`Ctrl+b`)

Эмуляция привычных tmux-биндов:

| Бинд | Действие |
|---|---|
| `"` | Split вниз |
| `%` | Split вправо |
| `c` | Новая вкладка |
| `n` / `p` | Следующая / предыдущая вкладка |
| `,` | Переименовать вкладку |
| `o` | Следующая панель |
| `z` | Fullscreen |
| `[` | Перейти в scroll |
| `space` | Следующий swap-layout |
| `d` | Detach |
| `h/j/k/l` или стрелки | Фокус по направлению |

---

## Сессии

Сериализация **включена** (`session_serialization true`), поэтому сессии переживают:
- закрытие окна терминала,
- ребут (если сериализация успела сработать — по умолчанию каждые 10 секунд: `serialization_interval`),
- завершение демона zellij.

При перезапуске сессии с тем же именем (`zellij a <name>` или `zellij --session <name>`) — восстанавливаются вкладки, панели, cwd, **видимое содержимое буфера** (`serialize_pane_viewport true`).

```bash
zellij ls                            # активные + сериализованные
zellij a <name>                      # attach к сессии (создаёт, если нет)
zellij --session work                # запустить с именем
zellij kill-session <name>           # завершить и удалить
zellij delete-all-sessions           # очистить все сериализованные сессии
```

Сериализованные сессии лежат в:
`~/Library/Application Support/org.Zellij-Contributors.Zellij/`.

---

## Layouts

Layout — это шаблон стартового состояния: какие вкладки, как разбиты панели, какие команды в них запустить. Файлы — KDL, лежат в `~/.config/zellij/layouts/`.

Запуск:

```bash
zellij --layout dev
zellij --layout k8s
zellij --layout ops
```

Сделать дефолтным — раскомментировать в `config.kdl`:

```kdl
default_layout "dev"
```

### Что внутри:

**`dev.kdl`** — основной рабочий layout.
- `code` (фокусная) — editor (70% по ширине) + shell + git status.
- `run` — пустая большая панель под запуск приложения/тестов.
- `scratch` — для черновых команд.

**`k8s.kdl`** — для работы с Kubernetes.
- `k9s` (фокусная) — запуск `k9s`.
- `kubectl` — большая `kubectl`-панель + панель под логи (`stern`/`kubectl logs -f`).
- `contexts` — вывод `kubectl config get-contexts`.

**`ops.kdl`** — системное наблюдение.
- `monitor` (фокусная) — `htop` сверху, shell + логи снизу.
- `net` — две панели под `lsof`, `nc`, `dig` и т.п.
- `shell` — чистая панель.

### Создать свой layout

```kdl
layout {
    default_tab_template {
        children
        pane size=1 borderless=true {
            plugin location="zellij:status-bar"
        }
    }

    tab name="main" focus=true {
        pane split_direction="vertical" {
            pane size="60%" name="editor"
            pane split_direction="horizontal" {
                pane name="shell"
                pane name="tests" command="npm" {
                    args "test" "--" "--watch"
                }
            }
        }
    }
}
```

Полезные атрибуты:

- `pane split_direction="vertical|horizontal"`
- `pane size="50%"` или `size=20` (строк/колонок)
- `pane name="..."` — имя в заголовке
- `pane command="<cmd>" { args "a" "b" }` — автозапуск
- `pane cwd="/path"`
- `pane focus=true`
- `pane borderless=true`
- `floating_panes { pane { x 10; y 5; width 80; height 24 } }`

---

## Плагины

Подключены встроенные плагины (см. блок `plugins` в `config.kdl`):

- **about** — окно «О Zellij»
- **compact-bar** — компактная статус-строка (альтернатива `tab-bar` + `status-bar`)
- **configuration** — TUI редактор настроек
- **filepicker / strider** — файловый браузер (cwd = `/`)
- **plugin-manager** — установка/удаление плагинов
- **session-manager** — список и переключение сессий
- **status-bar** — нижняя строка с подсказками режима
- **tab-bar** — верхняя строка вкладок
- **welcome-screen** — стартовый экран (на основе session-manager)
- **zellij:link** — обработчик OSC8 ссылок (грузится в фоне через `load_plugins`)

Установить сторонний:

```kdl
plugins {
    my-plugin location="https://example.com/plugin.wasm"
}

load_plugins {
    my-plugin
}
```

Перезапустить zellij для применения.

---

## Дефолтный dev-набор

В `~/.config/zellij/plugins/` уже лежат 5 сторонних плагинов и подключены через алиасы в `config.kdl`. Кратко — зачем каждый и как пользоваться.

### 1. zjstatus — статус-бар

Заменяет встроенный `tab-bar` + `status-bar` на одну строку с темой gruvbox.
Показывает: режим, имя сессии, вкладки, git-ветку, время (Europe/Moscow).

- В `dev.kdl` справа: `git branch + datetime`
- В `k8s.kdl` справа: `kubectl current-context + datetime`
- В `ops.kdl` справа: `load average (uptime) + datetime`

Виджеты конфигурируются в самом layout-файле, в секции `plugin location="zjstatus" { ... }`. Полный референс: [github.com/dj95/zjstatus/wiki](https://github.com/dj95/zjstatus/wiki).

### 2. zellij-sessionizer — пикер сессий по папкам

`Alt+g` → floating-окно со списком всех подкаталогов из `root_dirs`. Выбираешь — создаётся (или восстанавливается) сессия с именем папки, cwd в неё.

**Настрой под себя** в `config.kdl`, секция `sessionizer`:
```kdl
root_dirs "/Users/jtprogru/projects;/Users/jtprogru/dev;/Users/jtprogru/work"
individual_dirs "/Users/jtprogru/.config;/Users/jtprogru/.config/zellij"
```

- `root_dirs` — папки, в которых сканируются подкаталоги на одном уровне.
- `individual_dirs` — конкретные папки (без сканирования).

Аналог tmux-sessionizer от ThePrimeagen.

### 3. vim-zellij-navigator — бесшовная навигация с (n)vim

`Ctrl+h/j/k/l` — сначала пытается перейти к сплиту внутри vim/nvim, и только если уже на краю — переключается на соседнюю zellij-панель. То же в обратную сторону.

**Требует плагин в Neovim:**
```lua
-- lazy.nvim
{ "mrjones2014/smart-splits.nvim" }
```

И биндинги в nvim:
```lua
vim.keymap.set('n', '<C-h>', require('smart-splits').move_cursor_left)
vim.keymap.set('n', '<C-j>', require('smart-splits').move_cursor_down)
vim.keymap.set('n', '<C-k>', require('smart-splits').move_cursor_up)
vim.keymap.set('n', '<C-l>', require('smart-splits').move_cursor_right)
```

Альтернативы: `Navigator.nvim` (умеет и tmux, и zellij), `zellij-nav.nvim`.

**Важно:** старый `Ctrl+h` (вход в move-mode) переехал на `Alt+m`.

### 4. zellij-autolock — авто-locked при vim/fzf/k9s

Грузится в фоне (через `load_plugins`). Каждый раз когда нажимаешь `Enter` в normal-mode, плагин смотрит foreground-процесс активной панели — если он попадает в `triggers`, zellij переходит в locked-mode (отдаёт все кейкомбы приложению). Выходишь из приложения — автоматически возвращается в normal.

Сейчас в `triggers`:
```
nvim|vim|fzf|zoxide|lazygit|k9s|btop|htop|less|man
```

Зачем: чтобы кейбинды zellij (`Ctrl+p`, `Ctrl+s`, `Ctrl+n` и т.п.) не отбирались у этих приложений.

`Alt+z` — ручной toggle, если нужно отключить/включить.

### 5. harpoon — закладки панелей

`Ctrl+y` открывает floating-окно. Внутри:
- `a` — добавить текущую панель в закладки
- `A` — добавить все панели текущей сессии
- `j`/`k` или стрелки — навигация
- `Enter` / `l` — прыгнуть на выбранную панель
- `d` — удалить из списка
- `Esc` — выйти

Полезно, когда часто прыгаешь между 3-4 ключевыми панелями в большой сессии.

### Обновление плагинов

```bash
cd ~/.config/zellij/plugins
curl -fsSL -O https://github.com/dj95/zjstatus/releases/latest/download/zjstatus.wasm
curl -fsSL -O https://github.com/laperlej/zellij-sessionizer/releases/latest/download/zellij-sessionizer.wasm
curl -fsSL -O https://github.com/hiasr/vim-zellij-navigator/releases/latest/download/vim-zellij-navigator.wasm
curl -fsSL -O https://github.com/fresh2dev/zellij-autolock/releases/latest/download/zellij-autolock.wasm
curl -fsSL -O https://github.com/Nacho114/harpoon/releases/latest/download/harpoon.wasm
```

Затем перезапустить все сессии (`zellij kill-all-sessions` → новый запуск).

---

## Тема

Подключена встроенная `gruvbox-dark`. Полный список встроенных тем:

```bash
zellij setup --dump-config | grep -A1 "^themes" | head -30
```

Или просто перебирать значения в `theme "..."` — допустимы:
`default`, `dracula`, `gruvbox-dark`, `gruvbox-light`, `solarized-dark`, `solarized-light`,
`catppuccin-latte`, `catppuccin-frappe`, `catppuccin-macchiato`, `catppuccin-mocha`,
`tokyo-night`, `tokyo-night-dark`, `tokyo-night-light`, `tokyo-night-storm`,
`one-half-dark`, `nord`, `monokai-dark`, `monokai-soft`, `everforest-dark`, `everforest-light`.

Кастомная тема — добавить блок `themes` в `config.kdl`:

```kdl
themes {
    my-theme {
        fg "#ebdbb2"
        bg "#282828"
        black "#282828"
        red "#cc241d"
        green "#98971a"
        yellow "#d79921"
        blue "#458588"
        magenta "#b16286"
        cyan "#689d6a"
        white "#a89984"
        orange "#d65d0e"
    }
}

theme "my-theme"
```

Парные `theme_dark` / `theme_light` — автопереключение по системной палитре терминала.

---

## Полезные команды CLI

```bash
zellij                                  # запуск
zellij --layout <name>                  # с указанным layout
zellij -s <name>                        # с именем сессии
zellij a <name>                         # attach
zellij a --create <name>                # attach или создать
zellij ls                               # список сессий
zellij kill-session <name>
zellij kill-all-sessions
zellij delete-session <name>            # удалить сериализованные данные
zellij delete-all-sessions

zellij setup --check                    # проверить конфиг
zellij setup --dump-config              # вывести дефолтный конфиг (для референса)
zellij setup --dump-layout default      # вывести дефолтный layout
zellij options --simplified-ui true     # запуск с UI без спец-шрифтов
zellij options --disable-mouse-mode

# Запустить команду в новой панели текущей сессии (изнутри другой панели)
zellij action new-pane -- htop
zellij action new-tab --name logs
zellij action write-chars "echo hello"
zellij action go-to-tab 2
```

---

## Решение проблем

**Иконки/стрелки в статус-баре «битые»** — терминал без Nerd Font. Варианты:
- поставить Nerd Font в терминал (`brew install --cask font-jetbrains-mono-nerd-font`);
- или включить упрощённый UI: `simplified_ui true` в `config.kdl`.

**Мышь мешает выделять текст для копирования** — `mouse_mode false` или зажимать `Shift` при выделении.

**Не работают Alt-бинды** — терминал перехватывает Alt как Meta. В iTerm2: *Profiles → Keys → Left Option Key = Esc+* (или Normal, но тогда меняй бинды).
В Ghostty/Alacritty/Kitty Alt обычно работает из коробки.

**После апдейта zellij конфиг ломается** — `zellij setup --check` укажет на проблему. Сравни с актуальным `zellij setup --dump-config`.

**Хочу tmux-бинды** — они уже есть через `Ctrl+b` (см. раздел [tmux mode](#tmux-mode-ctrlb)).

**Сессия не восстановилась после ребута** — `serialization_interval` по умолчанию 10 сек. Если приложение было закрыто раньше — сериализация не успела. Уменьшить интервал в `config.kdl`:
```kdl
serialization_interval 5
```

**Конфиг полностью сбросить к дефолту** —
```bash
zellij setup --dump-config > ~/.config/zellij/config.kdl
```

---

## Ссылки

- Документация: https://zellij.dev/documentation/
- Конфиг-референс: https://zellij.dev/documentation/configuration
- Layouts: https://zellij.dev/documentation/layouts
- Плагины: https://zellij.dev/documentation/plugins
- KDL формат: https://kdl.dev
