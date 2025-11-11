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

### Шаблоны React Компонентов

#### 7. Шаблон корня документа - `document_root_template()`

**Расположение**: `reflex/compiler/templates.py:145`

**Описание**: Генерирует корневой компонент документа (Layout), который оборачивает все приложение. Это компонент верхнего уровня, который определяет структуру HTML документа.

**Параметры**:
- `imports` (list) - список импортов для компонента
- `document` (dict) - структура корневого компонента документа

**Использование**: Создается при компиляции для определения HTML структуры приложения.

**Код** (сокращенная версия):
```python
def document_root_template(*, imports: list[_ImportDict], document: dict[str, Any]):
    """Template for the document root.

    Generates Layout component with imports and document structure.
    """
```

---

#### 8. Шаблон корня приложения - `app_root_template()`

**Расположение**: `reflex/compiler/templates.py:165`

**Описание**: Генерирует корневой компонент приложения (_app.js), который содержит провайдеры, хуки и общую логику приложения. Это точка входа для всех страниц.

**Параметры**:
- `imports` (list) - список импортов
- `custom_codes` (iterable) - кастомный JavaScript код
- `hooks` (dict) - React хуки (useEffect, useState и т.д.)
- `window_libraries` (list) - библиотеки для глобального объекта window
- `render` (dict) - структура рендера компонента
- `dynamic_imports` (set) - динамические импорты

**Использование**: Создается при компиляции как главный компонент приложения.

**Генерируемые элементы**:
- `AppWrap` - компонент с хуками и рендером
- `Layout` - компонент с провайдерами (StateProvider, EventLoopProvider, ThemeProvider)
- `App` - главный компонент приложения
- Регистрация библиотек в `window.__reflex`

**Код** (сокращенная версия):
```python
def app_root_template(
    *,
    imports: list[_ImportDict],
    custom_codes: Iterable[str],
    hooks: dict[str, VarData | None],
    window_libraries: list[tuple[str, str]],
    render: dict[str, Any],
    dynamic_imports: set[str],
):
    """Template for the App root.

    Generates main App component with providers, hooks, and global setup.
    """
```

---

#### 9. Шаблон страницы - `page_template()`

**Расположение**: `reflex/compiler/templates.py:429`

**Описание**: Генерирует React компонент для отдельной страницы приложения. Каждая страница имеет свои импорты, хуки и рендер логику.

**Параметры**:
- `imports` (iterable) - список импортов для страницы
- `dynamic_imports` (iterable) - динамические импорты
- `custom_codes` (iterable) - кастомный код
- `hooks` (dict) - React хуки страницы
- `render` (dict) - структура рендера

**Использование**: Создается для каждой страницы приложения при компиляции.

**Код**:
```python
def page_template(
    imports: Iterable[_ImportDict],
    dynamic_imports: Iterable[str],
    custom_codes: Iterable[str],
    hooks: dict[str, VarData | None],
    render: dict[str, Any],
):
    """Template for a single react page.

    Returns:
        Rendered React page component as string.
    """
    imports_str = "\n".join([_RenderUtils.get_import(imp) for imp in imports])
    custom_code_str = "\n".join(custom_codes)
    dynamic_imports_str = "\n".join(dynamic_imports)

    hooks_str = _render_hooks(hooks)
    return f"""{imports_str}

{dynamic_imports_str}

{custom_code_str}

export default function Component() {{
{hooks_str}

  return (
    {_RenderUtils.render(render)}
  )
}}"""
```

---

#### 10. Шаблон компонента - `component_template()`

**Расположение**: `reflex/compiler/templates.py:417`

**Описание**: Генерирует рендер отдельного компонента. Это базовый шаблон для преобразования Reflex компонента в React JSX код.

**Параметры**:
- `component` (Component | StatefulComponent) - компонент для рендера

**Использование**: Используется внутренне при компиляции каждого Reflex компонента в React.

**Код**:
```python
def component_template(component: Component | StatefulComponent):
    """Template to render a component tag.

    Args:
        component: The component to render.

    Returns:
        Rendered component as string.
    """
    return _RenderUtils.render(component.render())
```

---

#### 11. Шаблон stateful компонента - `stateful_component_template()`

**Расположение**: `reflex/compiler/templates.py:610`

**Описание**: Генерирует stateful компонент - компонент с внутренним состоянием и хуками. Используется для компонентов, которые имеют собственную логику и состояние.

**Параметры**:
- `tag_name` (str) - имя компонента
- `memo_trigger_hooks` (list) - хуки для мемоизации
- `component` (Component) - компонент для рендера
- `export` (bool) - экспортировать ли компонент

**Использование**: Создается для компонентов с состоянием при компиляции.

**Код**:
```python
def stateful_component_template(
    tag_name: str, memo_trigger_hooks: list[str], component: Component, export: bool
):
    """Template for stateful component.

    Returns:
        Rendered stateful component code as string.
    """
    all_hooks = component._get_all_hooks()
    return f"""
{"export " if export else ""}function {tag_name} () {{
  {_render_hooks(all_hooks, memo_trigger_hooks)}
  return (
    {_RenderUtils.render(component.render())}
  )
}}
"""
```

---

#### 12. Шаблон memo компонентов - `memo_components_template()`

**Расположение**: `reflex/compiler/templates.py:649`

**Описание**: Генерирует мемоизированные компоненты используя React.memo для оптимизации перерисовок. Компоненты пересоздаются только при изменении их props.

**Параметры**:
- `imports` (list) - импорты
- `components` (list) - список компонентов для мемоизации
- `dynamic_imports` (iterable) - динамические импорты
- `custom_codes` (iterable) - кастомный код

**Использование**: Создается для оптимизации производительности часто используемых компонентов.

**Код** (сокращенная версия):
```python
def memo_components_template(
    imports: list[_ImportDict],
    components: list[dict[str, Any]],
    dynamic_imports: Iterable[str],
    custom_codes: Iterable[str],
) -> str:
    """Template for memoized components.

    Generates components wrapped in React.memo for performance optimization.
    """
```

---

## Шаблоны Кастомных Компонентов

Шаблоны для создания и публикации собственных Reflex компонентов как отдельных Python пакетов.

**Расположение**: `reflex/custom_components/custom_components.py`

### 1. Шаблон pyproject.toml - `_pyproject_toml_template()`

**Расположение**: `reflex/custom_components/custom_components.py:21`

**Описание**: Генерирует файл `pyproject.toml` для кастомного компонента. Этот файл определяет метаданные пакета, зависимости и настройки сборки для публикации в PyPI.

**Параметры**:
- `package_name` (str) - имя пакета для PyPI
- `module_name` (str) - имя Python модуля
- `reflex_version` (str) - минимальная версия Reflex

**Использование**: Создается при инициализации нового кастомного компонента через `reflex component init`.

**Генерируемая структура**:
- Build system: setuptools, wheel
- Метаданные проекта: имя, версия, описание, лицензия
- Зависимости: минимальная версия Reflex
- Опциональные зависимости для разработки: build, twine

**Код** (сокращенная версия):
```python
def _pyproject_toml_template(
    package_name: str, module_name: str, reflex_version: str
) -> str:
    """Template for custom components pyproject.toml.

    Args:
        package_name: The name of the package.
        module_name: The name of the module.
        reflex_version: The version of Reflex.

    Returns:
        Rendered pyproject.toml content as string.
    """
    return f"""[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "{package_name}"
version = "0.0.1"
description = "Reflex custom component {module_name}"
readme = "README.md"
license = {{ text = "Apache-2.0" }}
requires-python = ">=3.10"
authors = [{{ name = "", email = "YOUREMAIL@domain.com" }}]
keywords = ["reflex","reflex-custom-components"]

dependencies = ["reflex>={reflex_version}"]

classifiers = ["Development Status :: 4 - Beta"]

[project.optional-dependencies]
dev = ["build", "twine"]

[tool.setuptools.packages.find]
where = ["custom_components"]
"""
```

---

### 2. Шаблон README - `_readme_template()`

**Расположение**: `reflex/custom_components/custom_components.py:62`

**Описание**: Генерирует README.md файл для кастомного компонента с базовой документацией и инструкциями по установке.

**Параметры**:
- `module_name` (str) - имя модуля компонента
- `package_name` (str) - имя пакета для установки

**Использование**: Создается автоматически при инициализации кастомного компонента.

**Генерируемая структура**:
- Заголовок с именем компонента
- Краткое описание
- Инструкции по установке через pip

**Код**:
```python
def _readme_template(module_name: str, package_name: str) -> str:
    """Template for custom components README.

    Args:
        module_name: The name of the module.
        package_name: The name of the package.

    Returns:
        Rendered README.md content as string.
    """
    return f"""# {module_name}

A Reflex custom component {module_name}.

## Installation

```bash
pip install {package_name}
```
"""
```

---

### 3. Шаблон исходного кода компонента - `_source_template()`

**Расположение**: `reflex/custom_components/custom_components.py:84`

**Описание**: Генерирует базовый исходный код для нового кастомного компонента. Включает шаблон класса компонента с комментариями и примерами использования различных функций.

**Параметры**:
- `component_class_name` (str) - имя класса компонента
- `module_name` (str) - имя модуля

**Использование**: Создается как отправная точка для разработки кастомного компонента.

**Генерируемые элементы**:
- Импорт Reflex
- Класс компонента, наследующий от `rx.Component`
- Комментарии с инструкциями по настройке:
  - `library` - React библиотека для обертки
  - `tag` - тег React компонента
  - `is_default` - флаг default export
  - `alias` - алиас для избежания конфликтов имен
  - Props компонента
  - `lib_dependencies` - дополнительные зависимости
  - Event triggers
  - Custom code
- Функция-хелпер для создания компонента

**Код** (полный):
```python
def _source_template(component_class_name: str, module_name: str) -> str:
    """Template for custom components source.

    Args:
        component_class_name: The name of the component class.
        module_name: The name of the module.

    Returns:
        Rendered custom component source code as string.
    """
    return rf'''
"""Reflex custom component {component_class_name}."""

# For wrapping react guide, visit https://reflex.dev/docs/wrapping-react/overview/

import reflex as rx

# Some libraries you want to wrap may require dynamic imports.
# This is because they they may not be compatible with Server-Side Rendering (SSR).
# To handle this in Reflex, all you need to do is subclass `NoSSRComponent` instead.
# For example:
# from reflex.components.component import NoSSRComponent
# class {component_class_name}(NoSSRComponent):
#     pass


class {component_class_name}(rx.Component):
    """{component_class_name} component."""

    # The React library to wrap.
    library = "Fill-Me"

    # The React component tag.
    tag = "Fill-Me"

    # If the tag is the default export from the module, you must set is_default = True.
    # This is normally used when components don't have curly braces around them when importing.
    # is_default = True

    # If you are wrapping another components with the same tag as a component in your project
    # you can use aliases to differentiate between them and avoid naming conflicts.
    # alias = "Other{component_class_name}"

    # The props of the React component.
    # Note: when Reflex compiles the component to Javascript,
    # `snake_case` property names are automatically formatted as `camelCase`.
    # The prop names may be defined in `camelCase` as well.
    # some_prop: rx.Var[str] = "some default value"
    # some_other_prop: rx.Var[int] = 1

    # By default Reflex will install the library you have specified in the library property.
    # However, sometimes you may need to install other libraries to use a component.
    # In this case you can use the lib_dependencies property to specify other libraries to install.
    # lib_dependencies: list[str] = []

    # Event triggers declaration if any.
    # Below is equivalent to merging `{{ "on_change": lambda e: [e] }}`
    # onto the default event triggers of parent/base Component.
    # The function defined for the `on_change` trigger maps event for the javascript
    # trigger to what will be passed to the backend event handler function.
    # on_change: rx.EventHandler[lambda e: [e]]

    # To add custom code to your component
    # def _get_custom_code(self) -> str:
    #     return "const customCode = 'customCode';"


{module_name} = {component_class_name}.create
'''
```

---

### 4. Шаблон __init__.py - `_init_template()`

**Расположение**: `reflex/custom_components/custom_components.py:155`

**Описание**: Генерирует файл `__init__.py` для модуля кастомного компонента. Обеспечивает экспорт всех элементов из основного файла компонента.

**Параметры**:
- `module_name` (str) - имя модуля для импорта

**Использование**: Создается автоматически для правильной структуры Python пакета.

**Код**:
```python
def _init_template(module_name: str) -> str:
    """Template for custom components __init__.py.

    Args:
        module_name: The name of the module.

    Returns:
        Rendered __init__.py content as string.
    """
    return f"from .{module_name} import *"
```

---

### 5. Шаблон демо приложения - `_demo_app_template()`

**Расположение**: `reflex/custom_components/custom_components.py:167`

**Описание**: Генерирует демонстрационное Reflex приложение для тестирования кастомного компонента. Приложение показывает как использовать компонент и позволяет проверить его функциональность.

**Параметры**:
- `custom_component_module_dir` (str) - путь к директории с компонентом
- `module_name` (str) - имя модуля компонента

**Использование**: Создается при инициализации кастомного компонента для быстрого тестирования.

**Генерируемые элементы**:
- Импорты Reflex и кастомного компонента
- Класс State для управления состоянием
- Функция `index()` - главная страница с примером использования компонента
- Создание и запуск приложения

**Код** (сокращенная версия):
```python
def _demo_app_template(custom_component_module_dir: str, module_name: str) -> str:
    """Template for custom components demo app.

    Args:
        custom_component_module_dir: The directory of the custom component module.
        module_name: The name of the module.

    Returns:
        Rendered demo app source code as string.
    """
    return rf'''
"""Welcome to Reflex! This file showcases the custom component in a basic app."""

from rxconfig import config

import reflex as rx

from {custom_component_module_dir} import {module_name}

filename = f"{{config.app_name}}/{{config.app_name}}.py"


class State(rx.State):
    """The app state."""
    pass

def index() -> rx.Component:
    return rx.center(
        rx.theme_panel(),
        rx.vstack(
            rx.heading("Welcome to Reflex!", size="9"),
            rx.text(
                "Test your custom component by editing ",
                rx.code(filename),
            ),
            # Use the custom component here
            {module_name}(),
        )
    )

app = rx.App()
app.add_page(index)
'''
```

---

## Заключение

Эта документация охватывает все промты и шаблоны, используемые в Reflex:

1. **CLI Интерактивные Промты** - для взаимодействия с пользователем через консоль
2. **Шаблоны Компилятора** - для генерации конфигураций и React кода
3. **Шаблоны Кастомных Компонентов** - для создания пользовательских компонентов

Важно отметить, что Reflex **не содержит AI промптов** для языковых моделей. Все "промты" в системе - это либо CLI промты для пользователя, либо шаблоны генерации кода.
