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
