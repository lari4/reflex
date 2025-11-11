# Reflex Prompts and Templates Documentation

Эта документация содержит полный список всех промптов и шаблонов, используемых в фреймворке Reflex.

**Важно**: Reflex не содержит AI промптов для языковых моделей (LLM). Все "промты" в этой документации относятся к:
- CLI интерактивным промптам (вопросы пользователю через консоль)
- Шаблонам генерации кода

## Содержание

1. [CLI Интерактивные Промты](#cli-интерактивные-промты)
2. [Шаблоны Компилятора](#шаблоны-компилятора)
3. [Шаблоны Кастомных Компонентов](#шаблоны-кастомных-компонентов)

---

## CLI Интерактивные Промты

### 1. Общий промт пользователя - `ask()`

**Расположение**: `reflex/utils/console.py:366`

**Описание**: Универсальная функция для запроса ввода от пользователя через командную строку. Использует библиотеку `rich.prompt.Prompt` для создания интерактивных промптов с опциональным списком вариантов выбора.

**Параметры**:
- `question` - текст вопроса для пользователя
- `choices` - список возможных вариантов ответа (опционально)
- `default` - значение по умолчанию (опционально)
- `show_choices` - показывать ли варианты выбора

**Использование**: Применяется для любого интерактивного взаимодействия с пользователем в CLI.

**Код**:
```python
def ask(
    question: str,
    choices: list[str] | None = None,
    default: str | None = None,
    show_choices: bool = True,
) -> str | None:
    """Takes a prompt question and optionally a list of choices
     and returns the user input.

    Args:
        question: The question to ask the user.
        choices: A list of choices to select from.
        default: The default option selected.
        show_choices: Whether to show the choices.

    Returns:
        A string with the user input.
    """
    return Prompt.ask(
        question, choices=choices, default=default, show_choices=show_choices
    )
```

---

### 2. Промт выбора шаблона приложения - `prompt_for_template_options()`

**Расположение**: `reflex/utils/templates.py:318`

**Описание**: Интерактивный промт для выбора шаблона приложения при инициализации нового Reflex проекта. Показывает пользователю список доступных шаблонов с их описаниями и позволяет выбрать один из них по индексу.

**Параметры**:
- `templates` - список объектов Template с информацией о доступных шаблонах

**Использование**: Вызывается при выполнении `reflex init` без указания конкретного шаблона.

**Варианты шаблонов**:
1. AI Builder - перенаправляет на онлайн AI-конструктор
2. Blank App - создает пустое приложение
3. Choose Templates - показывает список готовых шаблонов

**Код**:
```python
def prompt_for_template_options(templates: list[Template]) -> str:
    """Prompt the user to specify a template.

    Args:
        templates: The templates to choose from.

    Returns:
        The template name the user selects.

    Raises:
        SystemExit: If the user does not select a template.
    """
    # Show the user the URLs of each template to preview.
    console.print("\nGet started with a template:")

    # Prompt the user to select a template.
    for index, template in enumerate(templates):
        console.print(f"({index}) {template.description}")

    template = console.ask(
        "Which template would you like to use?",
        choices=[str(i) for i in range(len(templates))],
        show_choices=False,
        default="0",
    )

    if not template:
        console.error("No template selected.")
        raise SystemExit(1)

    try:
        template_index = int(template)
    except ValueError:
        console.error("Invalid template selected.")
        raise SystemExit(1) from None

    if template_index < 0 or template_index >= len(templates):
        console.error("Invalid template selected.")
        raise SystemExit(1)

    # Return the template.
    return templates[template_index].name
```

---

### 3. Промт опций инициализации - `get_init_cli_prompt_options()`

**Расположение**: `reflex/utils/templates.py:414`

**Описание**: Возвращает список стандартных опций для CLI промта при инициализации нового Reflex приложения. Эти опции используются функцией `prompt_for_template_options()`.

**Использование**: Автоматически вызывается при инициализации проекта для предоставления стандартных вариантов выбора.

**Возвращаемые опции**:
1. **AI Builder** - запускает бесплатный AI-конструктор
2. **Blank App** - создает пустое приложение Reflex
3. **Choose Templates** - показывает готовые шаблоны от команды Reflex

**Код**:
```python
def get_init_cli_prompt_options() -> list[Template]:
    """Get the CLI options for initializing a Reflex app.

    Returns:
        The CLI options.
    """
    return [
        Template(
            name=constants.Templates.AI,
            description="[bold]Try our free AI builder.",
            code_url="",
        ),
        Template(
            name=constants.Templates.DEFAULT,
            description="A blank Reflex app.",
            code_url="",
        ),
        Template(
            name=constants.Templates.CHOOSE_TEMPLATES,
            description="Premade templates built by the Reflex team.",
            code_url="",
        ),
    ]
```

---

## Шаблоны Компилятора

Шаблоны компилятора используются для генерации кода React приложения, конфигурационных файлов и других ресурсов.

**Расположение**: `reflex/compiler/templates.py`

### Конфигурационные шаблоны

#### 1. Шаблон конфигурации Reflex - `rxconfig_template()`

**Расположение**: `reflex/compiler/templates.py:125`

**Описание**: Генерирует файл `rxconfig.py` - основной конфигурационный файл для Reflex приложения. Этот файл содержит настройки приложения, включая имя приложения и список плагинов.

**Параметры**:
- `app_name` (str) - имя приложения

**Использование**: Автоматически создается при инициализации нового проекта (`reflex init`).

**Генерируемая структура**:
- Импорт модуля `reflex`
- Объект `Config` с именем приложения
- Настройка плагинов по умолчанию (Sitemap, TailwindV4)

**Код**:
```python
def rxconfig_template(app_name: str):
    """Template for the Reflex config file.

    Args:
        app_name: The name of the application.

    Returns:
        Rendered Reflex config file content as string.
    """
    return f"""import reflex as rx

config = rx.Config(
    app_name="{app_name}",
    plugins=[
        rx.plugins.SitemapPlugin(),
        rx.plugins.TailwindV4Plugin(),
    ]
)"""
```

---

#### 2. Шаблон package.json - `package_json_template()`

**Расположение**: `reflex/compiler/templates.py:468`

**Описание**: Генерирует файл `package.json` для Node.js части проекта. Содержит скрипты для сборки, зависимости npm пакетов и настройки переопределения версий.

**Параметры**:
- `scripts` (dict) - скрипты npm (build, dev, и т.д.)
- `dependencies` (dict) - зависимости проекта
- `dev_dependencies` (dict) - зависимости для разработки
- `overrides` (dict) - переопределения версий зависимостей

**Использование**: Автоматически создается при компиляции проекта для настройки frontend части.

**Генерируемая структура**:
- Имя пакета: "reflex"
- Тип модуля: "module" (ES modules)
- Скрипты, зависимости и настройки

**Код**:
```python
def package_json_template(
    scripts: dict[str, str],
    dependencies: dict[str, str],
    dev_dependencies: dict[str, str],
    overrides: dict[str, str],
):
    """Template for package.json.

    Args:
        scripts: The scripts to include in the package.json file.
        dependencies: The dependencies to include in the package.json file.
        dev_dependencies: The devDependencies to include in the package.json file.
        overrides: The overrides to include in the package.json file.

    Returns:
        Rendered package.json content as string.
    """
    return json.dumps({
        "name": "reflex",
        "type": "module",
        "scripts": scripts,
        "dependencies": dependencies,
        "devDependencies": dev_dependencies,
        "overrides": overrides,
    })
```

---

#### 3. Шаблон конфигурации Vite - `vite_config_template()`

**Расположение**: `reflex/compiler/templates.py:495`

**Описание**: Генерирует файл `vite.config.js` для настройки Vite - инструмента сборки frontend приложения. Включает настройки плагинов, HMR (Hot Module Replacement), sourcemaps и другие опции разработки и сборки.

**Параметры**:
- `base` (str) - базовый путь для приложения
- `hmr` (bool) - включить Hot Module Replacement
- `force_full_reload` (bool) - принудительная полная перезагрузка при изменениях
- `experimental_hmr` (bool) - экспериментальные функции HMR
- `sourcemap` (bool | "inline" | "hidden") - настройки sourcemap

**Использование**: Создается при компиляции для настройки процесса сборки и dev-сервера.

**Код** (сокращенная версия):
```python
def vite_config_template(
    base: str,
    hmr: bool,
    force_full_reload: bool,
    experimental_hmr: bool,
    sourcemap: bool | Literal["inline", "hidden"],
):
    """Template for vite.config.js.

    Returns:
        Rendered vite.config.js content as string with plugins configuration,
        build settings, server options, and resolve aliases.
    """
    # Генерирует полный конфиг Vite с плагинами reactRouter, safariCacheBust
    # настройками сборки, HMR, sourcemaps и alias'ами для путей
```

---

#### 4. Шаблон контекста приложения - `context_template()`

**Расположение**: `reflex/compiler/templates.py:257`

**Описание**: Генерирует файл контекста React приложения с начальным состоянием, провайдерами состояния, обработчиками событий и настройками клиентского хранилища (localStorage/cookies).

**Параметры**:
- `is_dev_mode` (bool) - режим разработки
- `default_color_mode` (str) - цветовая схема по умолчанию
- `initial_state` (dict | None) - начальное состояние приложения
- `state_name` (str | None) - имя корневого состояния
- `client_storage` (dict | None) - настройки клиентского хранилища

**Использование**: Создается при компиляции для управления состоянием приложения на frontend.

**Генерируемые элементы**:
- `initialState` - начальное состояние всех state классов
- `StateProvider` - провайдер для управления состоянием
- `EventLoopProvider` - провайдер для обработки событий
- `UploadFilesProvider` - провайдер для загрузки файлов
- `initialEvents` - события, вызываемые при подключении WebSocket
- `onLoadInternalEvent` - события при загрузке страницы

**Код** (сокращенная версия):
```python
def context_template(
    *,
    is_dev_mode: bool,
    default_color_mode: str,
    initial_state: dict[str, Any] | None = None,
    state_name: str | None = None,
    client_storage: dict[str, dict[str, dict[str, Any]]] | None = None,
):
    """Template for the context file.

    Generates React context with state management, event loop,
    and client storage synchronization.
    """
```

---

#### 5. Шаблон темы - `theme_template()`

**Расположение**: `reflex/compiler/templates.py:245`

**Описание**: Генерирует файл темы приложения - простой экспорт объекта темы в формате JavaScript/TypeScript.

**Параметры**:
- `theme` (str) - JSON строка с настройками темы

**Использование**: Создается при компиляции для определения визуальной темы приложения.

**Код**:
```python
def theme_template(theme: str):
    """Template for the theme file.

    Args:
        theme: The theme to render.

    Returns:
        Rendered theme file content as string.
    """
    return f"""export default {theme}"""
```

---

#### 6. Шаблон стилей - `styles_template()`

**Расположение**: `reflex/compiler/templates.py:691`

**Описание**: Генерирует файл `styles.css` с импортами всех необходимых таблиц стилей. Использует `@layer` директиву для управления каскадом CSS и импортирует внешние стили через `@import`.

**Параметры**:
- `stylesheets` (list[str]) - список путей к таблицам стилей

**Использование**: Создается при компиляции для объединения всех CSS стилей приложения.

**Генерируемая структура**:
- Базовый layer `__reflex_base`
- Импорты всех указанных таблиц стилей

**Код**:
```python
def styles_template(stylesheets: list[str]) -> str:
    """Template for styles.css.

    Args:
        stylesheets: List of stylesheets to include.

    Returns:
        Rendered styles.css content as string.
    """
    return "@layer __reflex_base;\n" + "\n".join([
        f"@import url('{sheet_name}');" for sheet_name in stylesheets
    ])
```

---
