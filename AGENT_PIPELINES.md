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

## Пайплайн Обработки Событий

### Описание

Пайплайн обработки событий управляет взаимодействием между frontend (React) и backend (Python) через WebSocket соединение. Обрабатывает клики, ввод данных и другие пользовательские действия.

### Схема процесса

```
┌─────────────────────────────────────────┐
│  Пользователь кликает кнопку в UI       │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Frontend: React Event Handler          │
│  onClick={() => addEvents([             │
│    Event("state.increment")             │
│  ])}                                    │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  EventLoopContext.addEvents()           │
│  Добавление события в очередь           │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  WebSocket отправка на backend          │
│                                         │
│  Формат сообщения:                      │
│  {                                      │
│    "type": "event",                     │
│    "name": "state.increment",           │
│    "payload": {},                       │
│    "client_id": "abc123"                │
│  }                                      │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Backend: WebSocket сервер получает     │
│  сообщение                              │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Парсинг события                        │
│  - Извлечение имени: "state.increment"  │
│  - Разделение на state и метод          │
│    └─> state_name = "state"            │
│    └─> method_name = "increment"       │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Получение экземпляра State             │
│  state_instance = state_manager         │
│                   .get_state("state")   │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Вызов метода State                     │
│  state_instance.increment()             │
│                                         │
│  Python код:                            │
│  def increment(self):                   │
│      self.count += 1                    │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  State изменился                        │
│  state.count: 0 → 1                     │
│                                         │
│  Вычисление delta (изменений):         │
│  delta = {"count": 1}                   │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  WebSocket отправка delta на frontend   │
│                                         │
│  Формат ответа:                         │
│  {                                      │
│    "type": "state_update",              │
│    "delta": {                           │
│      "state": {"count": 1}              │
│    },                                   │
│    "events": []                         │
│  }                                      │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Frontend: Получение delta              │
│  EventLoop обрабатывает обновление      │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  React useReducer: applyDelta()         │
│                                         │
│  Применение изменений к state:          │
│  prevState.count = 0                    │
│  newState.count = 1                     │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  React re-render                        │
│  Обновление UI с новым значением        │
│  "Count: 1"                             │
└─────────────────────────────────────────┘
```

### Передаваемые данные

```
Frontend Event → Backend Processing → Frontend Update

1. User Action
   └─> onClick event
       └─> addEvents([Event("state.increment")])

2. WebSocket Message (Frontend → Backend)
   {
     "type": "event",
     "name": "state.increment",
     "payload": {},
     "client_id": "abc-123"
   }

3. Backend Processing
   ├─ Parse: "state" + "increment"
   ├─ Get State: state_manager.get_state("state")
   ├─ Call Method: state.increment()
   └─ Calculate Delta: {"count": 0 → 1}

4. WebSocket Response (Backend → Frontend)
   {
     "type": "state_update",
     "delta": {
       "state": {"count": 1}
     }
   }

5. Frontend State Update
   └─> applyDelta(prevState, delta)
       └─> newState = {...prevState, count: 1}
           └─> React re-render
```

### Используемые шаблоны

Генерируются во время компиляции:

1. **context_template()** - создает:
   - `EventLoopProvider` - провайдер для обработки событий
   - `addEvents` функция - добавление событий в очередь
   - `initialEvents` - события при загрузке
   - `onLoadInternalEvent` - внутренние события

2. **app_root_template()** - создает:
   - Интеграцию EventLoopProvider с приложением
   - Глобальный доступ к `addEvents` через Context

### Пример с параметрами события

```python
# Python
def set_name(self, new_name: str):
    self.name = new_name

# React (сгенерированный код)
<Input onChange={(e) => addEvents([
  Event("state.set_name", {new_name: e.target.value})
])} />
```

**Поток данных**:
```
1. Input onChange → e.target.value = "John"
2. Event("state.set_name", {new_name: "John"})
3. WebSocket → {"name": "state.set_name", "payload": {"new_name": "John"}}
4. Backend → state.set_name("John")
5. state.name = "John"
6. Delta → {"name": "John"}
7. Frontend update → state.name = "John"
8. Re-render с новым значением
```

---

## Пайплайн Рендеринга Компонентов

### Описание

Пайплайн рендеринга описывает, как Python компоненты Reflex преобразуются в React JSX во время компиляции и как они рендерятся в браузере.

### Схема процесса

```
┌─────────────────────────────────────────┐
│  Python: Определение компонента         │
│                                         │
│  def index():                           │
│      return rx.vstack(                  │
│          rx.heading("Title"),           │
│          rx.cond(                       │
│              State.show,                │
│              rx.text("Visible"),        │
│              rx.text("Hidden")          │
│          ),                             │
│          rx.foreach(                    │
│              State.items,               │
│              lambda item: rx.text(item) │
│          )                              │
│      )                                  │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Компиляция: component_template()       │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Обход дерева компонентов               │
│  _RenderUtils.render()                  │
└──────────────────┬──────────────────────┘
                   │
        ┌──────────┴──────────┐
        │  Тип компонента?    │
        └──────────┬──────────┘
                   │
    ┌──────────────┼──────────────┬──────────────┐
    │              │              │              │
    v              v              v              v
┌────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ Tag    │   │ Cond     │   │ Foreach  │   │ Match    │
│        │   │          │   │          │   │          │
└───┬────┘   └─────┬────┘   └─────┬────┘   └─────┬────┘
    │              │              │              │
    v              v              v              v
┌────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│render_tag()│ │render_       │ │render_       │ │render_       │
│            │ │condition_    │ │iterable_     │ │match_        │
│            │ │tag()         │ │tag()         │ │tag()         │
└─────┬──────┘ └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
      │               │                │                │
      v               v                v                v
┌─────────────────────────────────────────────────────────────┐
│  Генерация JSX кода                                         │
│                                                             │
│  1. rx.vstack → jsx(Flex, {direction: "column"}, ...)      │
│                                                             │
│  2. rx.heading → jsx(Heading, {}, "Title")                 │
│                                                             │
│  3. rx.cond → (State.show ? jsx(Text, {}, "Visible")       │
│                           : jsx(Text, {}, "Hidden"))        │
│                                                             │
│  4. rx.foreach → Array.prototype.map.call(                 │
│                    State.items ?? [],                       │
│                    (item, i) => jsx(Text, {}, item)         │
│                  )                                          │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Сгенерированный React компонент        │
│                                         │
│  export default function Component() {  │
│    const state = useContext(State);     │
│    return (                             │
│      jsx(Flex, {direction: "column"},   │
│        jsx(Heading, {}, "Title"),       │
│        (state.show ?                    │
│          jsx(Text, {}, "Visible") :     │
│          jsx(Text, {}, "Hidden")        │
│        ),                               │
│        Array.prototype.map.call(        │
│          state.items ?? [],             │
│          (item, i) => jsx(Text,{},item) │
│        )                                │
│      )                                  │
│    )                                    │
│  }                                      │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  Браузер: React выполнение              │
│  - Вызов Component()                    │
│  - Получение state из Context           │
│  - Вычисление условий (cond)            │
│  - Маппинг массивов (foreach)           │
│  - Генерация Virtual DOM                │
└──────────────────┬──────────────────────┘
                   │
                   v
┌─────────────────────────────────────────┐
│  React DOM рендеринг                    │
│  Virtual DOM → Actual DOM               │
│  Отображение в браузере                 │
└─────────────────────────────────────────┘
```

### Типы рендеринга

#### 1. Простой тег - `render_tag()`

```python
rx.heading("Hello", size="9")
```

**Компиляция**:
```
name: "Heading"
props: {size: "9"}
children: ["Hello"]

↓

jsx(Heading, {size: "9"}, "Hello")
```

#### 2. Условный рендеринг - `render_condition_tag()`

```python
rx.cond(State.is_logged_in,
    rx.text("Welcome"),
    rx.text("Please login")
)
```

**Компиляция**:
```
cond_state: "state.is_logged_in"
true_value: jsx(Text, {}, "Welcome")
false_value: jsx(Text, {}, "Please login")

↓

(state.is_logged_in ?
  jsx(Text, {}, "Welcome") :
  jsx(Text, {}, "Please login")
)
```

#### 3. Итерация - `render_iterable_tag()`

```python
rx.foreach(State.users, lambda user: rx.text(user.name))
```

**Компиляция**:
```
iterable_state: "state.users"
arg_name: "user"
arg_index: "i"
children: jsx(Text, {}, user.name)

↓

Array.prototype.map.call(
  state.users ?? [],
  (user, i) => jsx(Text, {}, user.name)
)
```

#### 4. Pattern matching - `render_match_tag()`

```python
rx.match(State.status,
    ("loading", rx.spinner()),
    ("success", rx.text("Done")),
    ("error", rx.text("Failed")),
    rx.text("Unknown")  # default
)
```

**Компиляция**:
```
switch(JSON.stringify(state.status)) {
  case JSON.stringify("loading"):
    return jsx(Spinner, {});
    break;
  case JSON.stringify("success"):
    return jsx(Text, {}, "Done");
    break;
  case JSON.stringify("error"):
    return jsx(Text, {}, "Failed");
    break;
  default:
    return jsx(Text, {}, "Unknown");
    break;
}
```

### Используемые шаблоны

1. **page_template()** - обертка для страницы с импортами и хуками
2. **component_template()** - рендеринг отдельного компонента
3. **_RenderUtils.render()** - главная функция рендеринга
4. **_RenderUtils.render_tag()** - обычные теги
5. **_RenderUtils.render_condition_tag()** - условия
6. **_RenderUtils.render_iterable_tag()** - циклы
7. **_RenderUtils.render_match_tag()** - pattern matching

### Полный пример: от Python к DOM

```python
# 1. Python код
def index():
    return rx.vstack(
        rx.heading(f"Count: {State.count}"),
        rx.button("+", on_click=State.increment)
    )
```

↓ **Компиляция**

```javascript
// 2. Сгенерированный React
export default function Component() {
  const [addEvents] = useContext(EventLoopContext);
  const state = useContext(StateContexts.State);

  return (
    jsx(Flex, {direction: "column"},
      jsx(Heading, {}, `Count: ${state.count}`),
      jsx(Button, {
        onClick: () => addEvents([Event("state.increment")])
      }, "+")
    )
  )
}
```

↓ **React выполнение**

```html
<!-- 3. Virtual DOM -->
<Flex direction="column">
  <Heading>Count: 5</Heading>
  <Button onClick={handler}>+</Button>
</Flex>
```

↓ **DOM рендеринг**

```html
<!-- 4. Actual DOM -->
<div class="rt-Flex rt-r-fd-column">
  <h1 class="rt-Heading rt-r-size-9">Count: 5</h1>
  <button class="rt-Button" onclick="...">+</button>
</div>
```

---

## Заключение

Эта документация описывает 4 основных пайплайна Reflex:

1. **Инициализация** - создание нового приложения
2. **Компиляция** - преобразование Python в React
3. **Обработка событий** - взаимодействие UI ↔ backend
4. **Рендеринг** - отображение компонентов в браузере

Все пайплайны используют шаблоны из `PROMPTS_DOCUMENTATION.md` для генерации кода.
