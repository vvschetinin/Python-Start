**Исправленная и дополненная инструкция**
Создание стартового учебного проекта **TypePython**
(Python 3.14 + uv + ruff + pytest, сентябрь 2026)

### Предварительные требования

- Windows (проверено на Git Bash / MINGW64 + PowerShell).
- Интернет.
- Папка проекта уже создана и вы находитесь внутри неё (`~/Desktop/TypePython`).

---

### Шаг 0. Установка uv (если ещё не установлен)

1. Откройте **PowerShell** (не Git Bash).
2. Выполните:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

3. Закройте PowerShell.
4. Откройте **новый** Git Bash и перейдите в папку проекта:

```bash
cd ~/Desktop/TypePython
```

5. Проверьте установку:

```bash
uv --version
```

Ожидаемый результат (версия может быть чуть новее):
`uv 0.12.19 ...`

> **Примечание.** Команда `curl ... | sh` в Git Bash часто падает с ошибкой `CRYPT_E_NO_REVOCATION_CHECK`. Используйте только PowerShell-способ.

---

### Шаг 1. Инициализация проекта

```bash
uv init --python 3.14 --no-package
```

Если появится ошибка «Project is already initialized» — это нормально (файл `pyproject.toml` уже существует). Просто продолжайте.

Проверьте содержимое:

```bash
ls -la
cat pyproject.toml
```

Должны быть файлы: `.python-version`, `main.py`, `pyproject.toml`, `README.md`.

---

### Шаг 2. Добавление dev-зависимостей

```bash
uv add --dev ruff pytest
```

Команда создаст `.venv`, `uv.lock` и добавит зависимости.

---

### Шаг 3. Настройка `pyproject.toml`

Полностью замените содержимое файла `pyproject.toml` на:

```toml
[project]
name = "typepython"
version = "0.1.0"
description = "Учебный проект «Месяц 1. Основы Python 3.14+ и современный toolchain»"
readme = "README.md"
requires-python = ">=3.14"
dependencies = []

[dependency-groups]
dev = [
    "ruff>=0.12.0",
    "pytest>=8.0",
]

[tool.ruff]
target-version = "py314"
line-length = 88
indent-width = 4

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "F",   # Pyflakes
    "I",   # isort
    "UP",  # pyupgrade
    "B",   # flake8-bugbear
    "SIM", # flake8-simplify
]
ignore = []

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
skip-magic-trailing-comma = false
line-ending = "auto"

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_functions = ["test_*"]
pythonpath = ["."]
```

Сохраните файл и выполните:

```bash
uv sync
```

---

### Шаг 4. Исходный код с type hints

Полностью замените содержимое `main.py` на:

```python
"""Простой пример: обработка списка чисел с type hints (Python 3.14)."""

from dataclasses import dataclass
from pathlib import Path


@dataclass(frozen=True, slots=True)
class NumberStats:
    """Статистика по списку чисел."""

    count: int
    total: float
    average: float


def compute_stats(numbers: list[float]) -> NumberStats:
    """Вычисляет количество, сумму и среднее."""
    if not numbers:
        return NumberStats(count=0, total=0.0, average=0.0)

    total = sum(numbers)
    count = len(numbers)
    average = total / count
    return NumberStats(count=count, total=total, average=average)


def load_numbers(path: Path) -> list[float]:
    """Читает числа из текстового файла (по одному на строку)."""
    text = path.read_text(encoding="utf-8")
    return [float(line.strip()) for line in text.splitlines() if line.strip()]


def main() -> None:
    """Точка входа."""
    data_file = Path("data.txt")

    # Создаём демо-файл, если его нет
    if not data_file.exists():
        data_file.write_text("10\n20.5\n30\n15.5\n", encoding="utf-8")

    numbers = load_numbers(data_file)
    stats = compute_stats(numbers)

    print(f"Файл: {data_file}")
    print(f"Количество: {stats.count}")
    print(f"Сумма: {stats.total:.2f}")
    print(f"Среднее: {stats.average:.2f}")


if __name__ == "__main__":
    main()
```

Проверка:

```bash
uv run python main.py
```

Ожидаемый вывод:

```
Файл: data.txt
Количество: 4
Сумма: 76.00
Среднее: 19.00
```

---

### Шаг 5. Базовые тесты

```bash
mkdir -p tests
```

Создайте файл `tests/test_main.py` со следующим содержимым:

```python
"""Базовые тесты для TypePython."""

from pathlib import Path

from main import NumberStats, compute_stats, load_numbers


def test_compute_stats_empty() -> None:
    result = compute_stats([])
    assert result == NumberStats(count=0, total=0.0, average=0.0)


def test_compute_stats_normal() -> None:
    result = compute_stats([10.0, 20.0, 30.0])
    assert result.count == 3
    assert result.total == 60.0
    assert result.average == 20.0


def test_load_numbers(tmp_path: Path) -> None:
    file = tmp_path / "nums.txt"
    file.write_text("1.5\n2.5\n3\n", encoding="utf-8")
    numbers = load_numbers(file)
    assert numbers == [1.5, 2.5, 3.0]
```

Запуск тестов:

```bash
uv run pytest -v
```

Ожидаемый результат: **3 passed**.

---

### Шаг 6. `.gitignore`

Создайте файл `.gitignore`:

```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
dist/
*.egg-info/
.eggs/

# Virtual environments
.venv/
venv/
ENV/

# Tools
.ruff_cache/
.pytest_cache/
.mypy_cache/
.coverage
htmlcov/

# IDE
.idea/
.vscode/
*.swp

# OS
.DS_Store
Thumbs.db

# Project specific
data.txt
```

---

### Шаг 7. README.md

Замените содержимое `README.md` на:

````markdown
# TypePython

Учебный стартовый проект «Месяц 1. Основы Python 3.14+ и современный toolchain».

## Быстрый старт

```bash
uv sync
uv run python main.py
uv run ruff check .
uv run ruff format .
uv run pytest
```
````

````

---

### Шаг 8. Финальная проверка

```bash
uv sync
uv run ruff check .
uv run ruff format --check .
uv run pytest -v
uv run python main.py
````

Все команды должны пройти без ошибок.

---

### Итоговая структура проекта

```
TypePython/
├── .gitignore
├── .python-version
├── .venv/                  # создаётся uv (не коммитить)
├── main.py
├── pyproject.toml
├── README.md
├── uv.lock
├── data.txt                # появляется при первом запуске
└── tests/
    └── test_main.py
```

---

### Список внесённых изменений и дополнений

1. **Добавлен Шаг 0** — полная инструкция по установке uv через PowerShell (основной рабочий способ на Windows).
2. **Добавлено предупреждение** о типичной ошибке `curl` + schannel (`CRYPT_E_NO_REVOCATION_CHECK`).
3. **Обработана ситуация**, когда проект уже частично инициализирован (`uv init` выдаёт ошибку «already initialized»).
4. **Исправлен критический баг импорта** в тестах: добавлен `pythonpath = ["."]` в секцию `[tool.pytest.ini_options]`.
5. **Уточнены команды проверки** на каждом этапе с ожидаемым результатом.
6. **Добавлены явные указания** «полностью заменить содержимое файла».
7. **Улучшена структура** инструкции: предварительные требования, нумерация шагов, итоговая структура.
8. **Сохранены все современные практики 2026**: `--no-package`, PEP 621, dependency-groups, ruff (lint + format), type hints + dataclasses + pathlib, Python 3.14.
9. **Убраны потенциальные источники ошибок** (нет Poetry, setup.py, src-layout, лишних папок).

Инструкция теперь воспроизводима с нуля и исключает все выявленные в процессе проблемы.
