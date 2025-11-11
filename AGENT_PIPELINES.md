# Reflex Pipelines Documentation

Эта документация описывает все основные пайплайны (потоки обработки) в фреймворке Reflex.

**Важно**: Reflex не является AI-агентом. Это фреймворк для создания веб-приложений. Документация описывает пайплайны обработки данных и генерации кода в рамках работы фреймворка.

## Содержание

1. [Пайплайн Инициализации Приложения](#пайплайн-инициализации-приложения)
2. [Пайплайн Компиляции](#пайплайн-компиляции)
3. [Пайплайн Обработки Событий](#пайплайн-обработки-событий)
4. [Пайплайн Рендеринга Компонентов](#пайплайн-рендеринга-компонентов)

---

## Пайплайн Инициализации Приложения

### Описание

Пайплайн инициализации запускается при выполнении команды `reflex init` и создает новое Reflex приложение с базовой структурой файлов и конфигурацией.

### Точка входа

`reflex/utils/templates.py:initialize_app()`

### Схема процесса

```
┌─────────────────────────────────────────┐
│  Пользователь запускает: reflex init    │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  initialize_app(app_name, template)     │
│  reflex/utils/templates.py:362          │
└──────────────────┬──────────────────────┘
                   │
                   v
         ┌────────┴────────┐
         │  Проверка:      │
         │  Существует ли  │
         │  rxconfig.py?   │
         └────────┬────────┘
                  │
         ┌────────┴────────┐
         │                 │
         v                 v
     ┌─────┐           ┌─────┐
     │ Да  │           │ Нет │
     └──┬──┘           └──┬──┘
        │                 │
        v                 v
   ┌─────────┐      ┌──────────────────────┐
   │ reinit  │      │ Запрос выбора        │
   │ teleme- │      │ шаблона (если не     │
   │ try     │      │ указан)              │
   └─────────┘      └──────────┬───────────┘
                               │
                               v
                    ┌──────────────────────┐
                    │ prompt_for_template_ │
                    │ options()            │
                    │ Промт пользователю   │
                    └──────────┬───────────┘
                               │
                               v
                    ┌──────────────────────┐
                    │ get_init_cli_prompt_ │
                    │ options()            │
                    │ Варианты:            │
                    │ 0. AI Builder        │
                    │ 1. Blank App         │
                    │ 2. Choose Templates  │
                    └──────────┬───────────┘
                               │
                               v
                    ┌──────────────────────┐
                    │ Выбор пользователя   │
                    └──────────┬───────────┘
                               │
            ┌──────────────────┼──────────────────┐
            │                  │                  │
            v                  v                  v
    ┌─────────────┐    ┌─────────────┐   ┌─────────────┐
    │ AI Builder  │    │ Blank App   │   │ Templates   │
    │ Redirect    │    │             │   │ (remote)    │
    └─────────────┘    └──────┬──────┘   └──────┬──────┘
                              │                  │
                              v                  v
                ┌──────────────────────┐  ┌──────────────────────┐
                │ initialize_default_  │  │ fetch_remote_        │
                │ app(app_name)        │  │ templates()          │
                └──────────┬───────────┘  └──────────┬───────────┘
                           │                         │
                           v                         v
                ┌──────────────────────┐  ┌──────────────────────┐
                │ create_config()      │  │ validate_and_create_ │
                │ Генерация:           │  │ app_using_remote_    │
                │ rxconfig.py          │  │ template()           │
                │                      │  │                      │
                │ Использует:          │  │ Загрузка ZIP файла   │
                │ rxconfig_template()  │  │ с GitHub             │
                └──────────┬───────────┘  └──────────┬───────────┘
                           │                         │
                           v                         v
                ┌──────────────────────┐  ┌──────────────────────┐
                │ initialize_app_      │  │ create_config_init_  │
                │ directory()          │  │ app_from_remote_     │
                │                      │  │ template()           │
                │ Копирование файлов   │  │                      │
                │ шаблона, переимено-  │  │ Распаковка ZIP,      │
                │ вание директорий и   │  │ копирование файлов,  │
                │ обновление импортов  │  │ переименование       │
                └──────────┬───────────┘  └──────────┬───────────┘
                           │                         │
                           └────────┬────────────────┘
                                    │
                                    v
                         ┌──────────────────────┐
                         │ Готовое приложение   │
                         │                      │
                         │ Структура:           │
                         │ - rxconfig.py        │
                         │ - app_name/          │
                         │   - app_name.py      │
                         │ - assets/            │
                         │ - requirements.txt   │
                         └──────────────────────┘
```

### Используемые промты и шаблоны

1. **CLI Промты**:
   - `ask()` - для получения ответов пользователя
   - `prompt_for_template_options()` - выбор типа шаблона
   - `get_init_cli_prompt_options()` - варианты шаблонов

2. **Шаблоны кода**:
   - `rxconfig_template(app_name)` - генерация конфигурационного файла

### Передаваемые данные между этапами

```
initialize_app
    ├─> app_name (str): имя приложения
    └─> template (str | None): выбранный шаблон
        │
        ├─> prompt_for_template_options
        │       └─> returns: template_name (str)
        │
        ├─> create_config
        │       ├─> input: app_name
        │       └─> generates: rxconfig.py file
        │
        └─> initialize_app_directory
                ├─> input: app_name, template_name, template_dir
                └─> generates: полная структура приложения
```

### Примеры выполнения

#### Пример 1: Создание пустого приложения

```bash
$ reflex init my_app --template blank
```

**Поток данных**:
1. `app_name = "my_app"`
2. `template = "blank"` (указан явно, промт пропускается)
3. Вызов `initialize_default_app("my_app")`
4. `create_config("my_app")` → генерация rxconfig.py
5. `initialize_app_directory("my_app")` → копирование шаблона
6. Результат: директория `my_app/` с базовым приложением

#### Пример 2: Интерактивный выбор шаблона

```bash
$ reflex init my_app
```

**Поток данных**:
1. `app_name = "my_app"`
2. `template = None` (не указан)
3. Вызов `prompt_for_template_options(get_init_cli_prompt_options())`
4. Промт пользователю:
   ```
   Get started with a template:
   (0) Try our free AI builder.
   (1) A blank Reflex app.
   (2) Premade templates built by the Reflex team.
   Which template would you like to use? [0]:
   ```
5. Пользователь выбирает: `1`
6. `template = "blank"`
7. Далее как в примере 1

---

## Пайплайн Компиляции

### Описание

Пайплайн компиляции преобразует Python код Reflex приложения в JavaScript/React код для frontend. Запускается при выполнении `reflex run` или `reflex export`.

### Точка входа

`reflex/compiler/compiler.py`

### Схема процесса

```
┌─────────────────────────────────────────┐
│  Пользователь запускает: reflex run     │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Загрузка rxconfig.py                   │
│  Чтение конфигурации приложения         │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Сбор всех компонентов приложения       │
│  - Страницы (pages)                     │
│  - Компоненты (components)              │
│  - State классы                         │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Этап 1: Генерация Context              │
│  context_template()                     │
│                                         │
│  Входные данные:                        │
│  - initial_state: dict всех State       │
│  - state_name: имя корневого State      │
│  - client_storage: настройки storage    │
│  - is_dev_mode: режим разработки        │
│  - default_color_mode: тема             │
│                                         │
│  Выходные данные:                       │
│  - .web/utils/context.js                │
│    ├─ initialState                      │
│    ├─ StateProvider                     │
│    ├─ EventLoopProvider                 │
│    └─ onLoadInternalEvent               │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Этап 2: Генерация Document Root        │
│  document_root_template()               │
│                                         │
│  Входные данные:                        │
│  - imports: список импортов             │
│  - document: структура HTML документа   │
│                                         │
│  Выходные данные:                       │
│  - .web/app/_document.js                │
│    └─ Layout компонент                  │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Этап 3: Генерация App Root             │
│  app_root_template()                    │
│                                         │
│  Входные данные:                        │
│  - imports: импорты библиотек           │
│  - custom_codes: кастомный JS код       │
│  - hooks: React хуки                    │
│  - window_libraries: глобальные библ.   │
│  - render: структура рендера            │
│  - dynamic_imports: динамические импорты│
│                                         │
│  Выходные данные:                       │
│  - .web/app/_app.js                     │
│    ├─ AppWrap компонент                 │
│    ├─ Layout с провайдерами             │
│    └─ App компонент                     │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Этап 4: Генерация страниц              │
│  page_template() для каждой страницы    │
│                                         │
│  Цикл для каждой страницы:              │
└──────────────────┬──────────────────────┘
                   │
           ┌───────┴────────┐
           │  Для каждой    │
           │  страницы:     │
           └───────┬────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Компиляция страницы                    │
│                                         │
│  Входные данные:                        │
│  - imports: импорты компонентов         │
│  - dynamic_imports: динамические импорты│
│  - custom_codes: кастомный код          │
│  - hooks: хуки страницы (useEffect и т.д.)│
│  - render: дерево компонентов           │
│                                         │
│  Процесс:                               │
│  1. Обход дерева компонентов            │
│  2. Для каждого компонента:             │
│     └─> component_template()            │
│         └─> _RenderUtils.render()       │
│             ├─ render_tag()             │
│             ├─ render_condition_tag()   │
│             ├─ render_iterable_tag()    │
│             └─ render_match_tag()       │
│                                         │
│  Выходные данные:                       │
│  - .web/pages/[page_name].js            │
│    └─ Component функция                 │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Этап 5: Генерация Stateful компонентов │
│  stateful_component_template()          │
│                                         │
│  Для компонентов с состоянием:         │
│  - memo_components_template()           │
│    для мемоизированных компонентов      │
│                                         │
│  Выходные данные:                       │
│  - .web/components/[comp_name].js       │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Этап 6: Генерация конфигурационных     │
│  файлов                                 │
│                                         │
│  - package_json_template()              │
│    → .web/package.json                  │
│                                         │
│  - vite_config_template()               │
│    → .web/vite.config.js                │
│                                         │
│  - theme_template()                     │
│    → .web/utils/theme.js                │
│                                         │
│  - styles_template()                    │
│    → .web/styles/styles.css             │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Этап 7: Установка npm зависимостей     │
│  npm install в .web/                    │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Готовое скомпилированное приложение    │
│                                         │
│  Структура .web/:                       │
│  - app/                                 │
│    ├─ _app.js                           │
│    └─ _document.js                      │
│  - pages/                               │
│    ├─ index.js                          │
│    └─ [other_pages].js                  │
│  - utils/                               │
│    ├─ context.js                        │
│    └─ theme.js                          │
│  - styles/                              │
│    └─ styles.css                        │
│  - package.json                         │
│  - vite.config.js                       │
│  - node_modules/                        │
└─────────────────────────────────────────┘
```

### Используемые промты и шаблоны

**Конфигурационные**:
1. `context_template()` - React контекст и state management
2. `package_json_template()` - npm конфигурация
3. `vite_config_template()` - конфигурация сборщика
4. `theme_template()` - тема приложения
5. `styles_template()` - CSS стили

**Компоненты**:
6. `document_root_template()` - корень документа
7. `app_root_template()` - корень приложения
8. `page_template()` - страницы
9. `component_template()` - базовые компоненты
10. `stateful_component_template()` - stateful компоненты
11. `memo_components_template()` - мемоизированные компоненты

### Передаваемые данные между этапами

```
Компиляция
├─> Этап 1: Context
│   ├─ input:
│   │   └─ app.state (State объект)
│   │       ├─ state_name: "State"
│   │       ├─ initial_state: {"State": {...}, "PageState": {...}}
│   │       └─ client_storage: {...}
│   └─ output: context.js
│       ├─ initialState
│       ├─ StateProvider
│       └─ initialEvents
│
├─> Этап 2: Document Root
│   ├─ input:
│   │   └─ document структура
│   └─ output: _document.js
│       └─ Layout компонент
│
├─> Этап 3: App Root
│   ├─ input:
│   │   ├─ global hooks
│   │   ├─ window libraries
│   │   └─ custom code
│   └─ output: _app.js
│       ├─ AppWrap
│       └─ App
│
├─> Этап 4: Pages (для каждой)
│   ├─ input:
│   │   └─ page component tree
│   │       ├─ rx.heading("Welcome")
│   │       ├─ rx.button("Click me", on_click=...)
│   │       └─ rx.vstack(children=[...])
│   │
│   ├─ process:
│   │   └─ component_template(component)
│   │       └─ _RenderUtils.render()
│   │           ├─ props extraction
│   │           ├─ children processing
│   │           └─ JSX generation
│   │
│   └─ output: page.js
│       └─ jsx(Component, {props}, ...children)
│
└─> Этап 5-7: Config files + npm install
```

### Пример преобразования компонента

#### Входной Python код:

```python
import reflex as rx

class State(rx.State):
    count: int = 0

    def increment(self):
        self.count += 1

def index():
    return rx.vstack(
        rx.heading("Counter App", size="9"),
        rx.text(f"Count: {State.count}"),
        rx.button("Increment", on_click=State.increment),
    )
```

#### Процесс компиляции:

```
index() компонент
    │
    ├─> rx.vstack(...)
    │   └─> component_template(vstack)
    │       └─> _RenderUtils.render_tag()
    │           ├─ name: "Flex"
    │           ├─ props: {direction: "column", ...}
    │           └─ children: [heading, text, button]
    │
    ├─> rx.heading(...)
    │   └─> component_template(heading)
    │       └─> _RenderUtils.render_tag()
    │           ├─ name: "Heading"
    │           ├─ props: {size: "9"}
    │           └─ children: ["Counter App"]
    │
    ├─> rx.text(...)
    │   └─> component_template(text)
    │       └─> _RenderUtils.render_tag()
    │           ├─ name: "Text"
    │           ├─ props: {}
    │           └─> Var обработка: State.count
    │               └─> {state.count} в JSX
    │
    └─> rx.button(...)
        └─> component_template(button)
            └─> _RenderUtils.render_tag()
                ├─ name: "Button"
                ├─ props: {}
                ├─ event handler: on_click
                │   └─> addEvents([Event("state.increment")])
                └─ children: ["Increment"]
```

#### Выходной JavaScript код:

```javascript
import {Flex, Heading, Text, Button} from "@radix-ui/themes"
import {jsx} from "@emotion/react"

export default function Component() {
  const [addEvents] = useContext(EventLoopContext);
  const state = useContext(StateContexts.State);

  return (
    jsx(Flex, {direction: "column"},
      jsx(Heading, {size: "9"}, "Counter App"),
      jsx(Text, {}, `Count: ${state.count}`),
      jsx(Button, {
        onClick: () => addEvents([Event("state.increment")])
      }, "Increment")
    )
  )
}
```

---
