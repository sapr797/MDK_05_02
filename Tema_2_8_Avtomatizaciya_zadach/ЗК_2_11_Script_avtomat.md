# ПЗ 2.11. Написание скрипта автоматизации сборки проекта

**Тема:** Основные конструкции языка программирования: переменные, ввод-вывод, автоматизация задач

**Цель работы:**  
Закрепить навыки использования переменных, ввода-вывода данных, условных конструкций и циклов на примере разработки скрипта автоматизации сборки проекта. Научиться создавать практичные утилиты командной строки для повседневных задач разработчика.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Git (опционально)

> Главная мысль: **Автоматизация начинается с малого — сначала вы пишете простой скрипт, который экономит 5 минут в день. Через месяц это уже часы сэкономленного времени.**

---

## Содержание

1. [Теоретическая справка](#1-теоретическая-справка)
2. [Нулевой вариант (эталонный)](#2-нулевой-вариант-эталонный)
3. [25 вариантов практической работы](#3-25-вариантов-практической-работы)
4. [Критерии оценки](#4-критерии-оценки)
5. [Шпаргалка](#5-шпаргалка)

---

## 1. Теоретическая справка

### 1.1. Что такое скрипт автоматизации сборки

**Скрипт сборки (build script)** — это программа, которая автоматически выполняет последовательность действий для подготовки проекта к запуску или деплою.

**Типичные задачи скрипта сборки:**

| Задача | Описание | Команда |
|--------|----------|---------|
| Проверка окружения | Убедиться, что установлен Python, pip, venv | `python --version` |
| Создание виртуального окружения | Изолированная среда для зависимостей | `python -m venv venv` |
| Активация окружения | Вход в виртуальную среду | `source venv/bin/activate` |
| Установка зависимостей | Установка библиотек из requirements.txt | `pip install -r requirements.txt` |
| Проверка качества кода | Линтеры и форматтеры | `pylint src/` |
| Запуск тестов | Проверка работоспособности | `pytest tests/` |
| Сборка пакета | Создание дистрибутива | `python setup.py sdist` |

### 1.2. Структура скрипта сборки

```python
#!/usr/bin/env python3
"""
build.py — Скрипт автоматизации сборки проекта.
"""

import os
import sys
import subprocess
from pathlib import Path

def run_command(cmd, description):
    """Запускает команду и выводит результат."""
    print(f"\n📦 {description}...")
    print(f"   Команда: {cmd}")
    
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    
    if result.returncode == 0:
        print(f"   ✅ {description} завершён")
        if result.stdout:
            print(result.stdout)
        return True
    else:
        print(f"   ❌ {description} не удался")
        print(result.stderr)
        return False

def main():
    """Основная функция скрипта."""
    print("=" * 50)
    print("🚀 ЗАПУСК СБОРКИ ПРОЕКТА")
    print("=" * 50)
    
    # Шаги сборки
    steps = [
        ("python --version", "Проверка версии Python"),
        ("pip --version", "Проверка pip"),
        ("python -m venv venv", "Создание виртуального окружения"),
    ]
    
    for cmd, desc in steps:
        run_command(cmd, desc)
    
    print("\n" + "=" * 50)
    print("✅ СБОРКА ЗАВЕРШЕНА")
    print("=" * 50)

if __name__ == "__main__":
    main()
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая разработку скрипта автоматизации сборки.

Техническое задание (нулевой вариант)
Разработайте скрипт build.py, который автоматизирует процесс сборки Python-проекта. Скрипт должен:

Проверять наличие Python и pip

Создавать виртуальное окружение venv

Активировать окружение и устанавливать зависимости из requirements.txt

Запускать линтер (flake8) для проверки кода

Запускать тесты (pytest)

Выводить цветной отчёт о выполнении

Поддерживать параметры командной строки: --skip-tests, --skip-lint, --clean

Эталонная реализация
python
#!/usr/bin/env python3
"""
build.py — Скрипт автоматизации сборки Python-проекта.

Использование:
    python build.py              # Полная сборка
    python build.py --skip-tests # Без тестов
    python build.py --skip-lint  # Без линтера
    python build.py --clean      # Очистка перед сборкой
    python build.py --help       # Справка
"""

import os
import sys
import subprocess
import argparse
import platform
from pathlib import Path
from datetime import datetime


# ============================================================
# ЦВЕТА ДЛЯ ВЫВОДА
# ============================================================

class Colors:
    """Цвета для терминала."""
    HEADER = '\033[95m'
    BLUE = '\033[94m'
    GREEN = '\033[92m'
    YELLOW = '\033[93m'
    RED = '\033[91m'
    END = '\033[0m'
    BOLD = '\033[1m'


def print_colored(text, color=Colors.GREEN):
    """Выводит цветной текст."""
    print(f"{color}{text}{Colors.END}")


# ============================================================
# ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ
# ============================================================

def run_command(cmd: str, description: str, silent: bool = False) -> bool:
    """
    Запускает команду и возвращает результат.

    Args:
        cmd: Команда для выполнения
        description: Описание шага
        silent: Подавлять вывод?

    Returns:
        True если успешно, False если ошибка
    """
    print(f"\n{Colors.BLUE}🔹 {description}{Colors.END}")
    if not silent:
        print(f"   Команда: {cmd}")

    try:
        if silent:
            result = subprocess.run(
                cmd, shell=True, capture_output=True, text=True
            )
        else:
            result = subprocess.run(cmd, shell=True, text=True)

        if result.returncode == 0:
            print_colored(f"   ✅ {description} завершён", Colors.GREEN)
            if result.stdout and not silent:
                print(result.stdout)
            return True
        else:
            print_colored(f"   ❌ {description} не удался", Colors.RED)
            if result.stderr and not silent:
                print(result.stderr)
            return False

    except Exception as e:
        print_colored(f"   ❌ Ошибка: {e}", Colors.RED)
        return False


def check_python_version() -> bool:
    """Проверяет версию Python."""
    version = sys.version_info
    if version.major >= 3 and version.minor >= 8:
        print_colored(f"   Python {version.major}.{version.minor}.{version.micro} — OK", Colors.GREEN)
        return True
    else:
        print_colored(f"   Python {version.major}.{version.minor} — требуется 3.8+", Colors.RED)
        return False


def get_venv_activate_command() -> str:
    """Возвращает команду активации виртуального окружения."""
    if platform.system() == "Windows":
        return "venv\\Scripts\\activate"
    else:
        return "source venv/bin/activate"


# ============================================================
# ОСНОВНЫЕ ШАГИ СБОРКИ
# ============================================================

def clean_project(project_path: Path) -> bool:
    """Очистка временных файлов."""
    print(f"\n{Colors.YELLOW}🧹 ОЧИСТКА ПРОЕКТА{Colors.END}")
    
    dirs_to_remove = ["__pycache__", ".pytest_cache", ".mypy_cache", "dist", "build", "*.egg-info"]
    files_to_remove = ["*.pyc", ".coverage", "coverage.xml"]
    
    success = True
    
    for pattern in dirs_to_remove:
        for path in project_path.glob(f"**/{pattern}"):
            if path.is_dir():
                print(f"   Удаление папки: {path}")
                try:
                    import shutil
                    shutil.rmtree(path)
                except Exception as e:
                    print_colored(f"   Ошибка: {e}", Colors.RED)
                    success = False
    
    for pattern in files_to_remove:
        for path in project_path.glob(f"**/{pattern}"):
            if path.is_file():
                print(f"   Удаление файла: {path}")
                try:
                    path.unlink()
                except Exception as e:
                    print_colored(f"   Ошибка: {e}", Colors.RED)
                    success = False
    
    return success


def create_venv(project_path: Path) -> bool:
    """Создаёт виртуальное окружение."""
    venv_path = project_path / "venv"
    
    if venv_path.exists():
        print_colored("   Виртуальное окружение уже существует", Colors.YELLOW)
        return True
    
    return run_command(
        f"{sys.executable} -m venv venv",
        "Создание виртуального окружения"
    )


def install_dependencies(project_path: Path, dev: bool = False) -> bool:
    """Устанавливает зависимости."""
    req_file = project_path / "requirements.txt"
    dev_req_file = project_path / "requirements-dev.txt"
    
    if not req_file.exists():
        print_colored("   requirements.txt не найден, пропускаем", Colors.YELLOW)
        return True
    
    activate_cmd = get_venv_activate_command()
    
    if dev and dev_req_file.exists():
        cmd = f"{activate_cmd} && pip install -r requirements.txt -r requirements-dev.txt"
    else:
        cmd = f"{activate_cmd} && pip install -r requirements.txt"
    
    return run_command(cmd, "Установка зависимостей")


def run_linter(project_path: Path) -> bool:
    """Запускает линтер flake8."""
    activate_cmd = get_venv_activate_command()
    cmd = f"{activate_cmd} && flake8 src --max-line-length=88 --count"
    
    return run_command(cmd, "Запуск линтера (flake8)")


def run_tests(project_path: Path, with_coverage: bool = False) -> bool:
    """Запускает тесты pytest."""
    activate_cmd = get_venv_activate_command()
    
    if with_coverage:
        cmd = f"{activate_cmd} && pytest --cov=src --cov-report=term"
    else:
        cmd = f"{activate_cmd} && pytest -v"
    
    return run_command(cmd, "Запуск тестов (pytest)")


def build_package(project_path: Path) -> bool:
    """Сборка пакета."""
    activate_cmd = get_venv_activate_command()
    
    # Проверяем наличие setup.py или pyproject.toml
    if (project_path / "setup.py").exists():
        cmd = f"{activate_cmd} && python setup.py sdist bdist_wheel"
    elif (project_path / "pyproject.toml").exists():
        cmd = f"{activate_cmd} && python -m build"
    else:
        print_colored("   Не найден setup.py или pyproject.toml", Colors.YELLOW)
        return True
    
    return run_command(cmd, "Сборка пакета")


def print_summary(results: dict, start_time: datetime) -> None:
    """Выводит итоговый отчёт."""
    print("\n" + "=" * 60)
    print_colored("ИТОГОВЫЙ ОТЧЁТ О СБОРКЕ", Colors.BOLD)
    print("=" * 60)
    
    for step, success in results.items():
        icon = "✅" if success else "❌"
        color = Colors.GREEN if success else Colors.RED
        print_colored(f"   {icon} {step}", color)
    
    end_time = datetime.now()
    duration = (end_time - start_time).total_seconds()
    
    print(f"\n⏱️ Время выполнения: {duration:.2f} сек")
    
    if all(results.values()):
        print_colored("\n🎉 СБОРКА ПРОШЛА УСПЕШНО!", Colors.GREEN)
    else:
        print_colored("\n⚠️ НЕКОТОРЫЕ ШАГИ НЕ УДАЛИСЬ!", Colors.RED)
        sys.exit(1)


# ============================================================
# ГЛАВНАЯ ФУНКЦИЯ
# ============================================================

def main():
    """Главная функция скрипта."""
    parser = argparse.ArgumentParser(
        description="Скрипт автоматизации сборки Python-проекта",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Примеры:
  python build.py               # Полная сборка
  python build.py --skip-tests  # Без тестов
  python build.py --skip-lint   # Без линтера
  python build.py --clean       # Очистка перед сборкой
  python build.py --dev         # Установка dev-зависимостей
        """
    )
    parser.add_argument("--skip-tests", action="store_true", help="Пропустить тесты")
    parser.add_argument("--skip-lint", action="store_true", help="Пропустить линтер")
    parser.add_argument("--clean", "-c", action="store_true", help="Очистка перед сборкой")
    parser.add_argument("--dev", "-d", action="store_true", help="Установка dev-зависимостей")
    parser.add_argument("--coverage", action="store_true", help="Запуск тестов с покрытием")
    
    args = parser.parse_args()
    
    # Определяем путь к проекту
    project_path = Path(__file__).parent.absolute()
    start_time = datetime.now()
    
    # Результаты шагов
    results = {}
    
    print("=" * 60)
    print_colored("🚀 ЗАПУСК СБОРКИ ПРОЕКТА", Colors.BOLD)
    print(f"   Путь: {project_path}")
    print(f"   Время: {start_time.strftime('%H:%M:%S')}")
    print("=" * 60)
    
    # Шаг 0: Очистка (опционально)
    if args.clean:
        results["Очистка"] = clean_project(project_path)
    
    # Шаг 1: Проверка окружения
    print(f"\n{Colors.BLUE}🔍 ПРОВЕРКА ОКРУЖЕНИЯ{Colors.END}")
    results["Python версия"] = check_python_version()
    
    # Шаг 2: Виртуальное окружение
    results["Виртуальное окружение"] = create_venv(project_path)
    
    # Шаг 3: Установка зависимостей
    results["Установка зависимостей"] = install_dependencies(project_path, args.dev)
    
    # Шаг 4: Линтер
    if not args.skip_lint:
        results["Линтер (flake8)"] = run_linter(project_path)
    else:
        results["Линтер (flake8)"] = True
        print_colored("\n⏭️ Линтер пропущен", Colors.YELLOW)
    
    # Шаг 5: Тесты
    if not args.skip_tests:
        results["Тесты (pytest)"] = run_tests(project_path, args.coverage)
    else:
        results["Тесты (pytest)"] = True
        print_colored("\n⏭️ Тесты пропущены", Colors.YELLOW)
    
    # Шаг 6: Сборка пакета
    results["Сборка пакета"] = build_package(project_path)
    
    # Итоговый отчёт
    print_summary(results, start_time)


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
============================================================
🚀 ЗАПУСК СБОРКИ ПРОЕКТА
   Путь: /home/user/my_project
   Время: 14:30:25
============================================================

🔍 ПРОВЕРКА ОКРУЖЕНИЯ
   Python 3.10.12 — OK

🔹 Создание виртуального окружения
   Команда: python3 -m venv venv
   ✅ Создание виртуального окружения завершён

🔹 Установка зависимостей
   Команда: source venv/bin/activate && pip install -r requirements.txt
   ✅ Установка зависимостей завершён

🔹 Запуск линтера (flake8)
   Команда: source venv/bin/activate && flake8 src --max-line-length=88 --count
   ✅ Запуск линтера (flake8) завершён

🔹 Запуск тестов (pytest)
   Команда: source venv/bin/activate && pytest -v
   ========= test session starts ==========
   tests/test_main.py::test_hello PASSED
   ========= 1 passed in 0.05s ===========
   ✅ Запуск тестов (pytest) завершён

🔹 Сборка пакета
   Команда: source venv/bin/activate && python -m build
   ✅ Сборка пакета завершён

============================================================
ИТОГОВЫЙ ОТЧЁТ О СБОРКЕ
============================================================
   ✅ Python версия
   ✅ Виртуальное окружение
   ✅ Установка зависимостей
   ✅ Линтер (flake8)
   ✅ Тесты (pytest)
   ✅ Сборка пакета

⏱️ Время выполнения: 12.34 сек

🎉 СБОРКА ПРОШЛА УСПЕШНО!
3. 25 вариантов практической работы
Общая структура каждого варианта:
Студент разрабатывает скрипт автоматизации сборки для заданного типа проекта. Скрипт должен использовать переменные, ввод-вывод, условные конструкции и циклы.

Уровни сложности:

Варианты 1-8: базовый (3-4 шага сборки)

Варианты 9-17: средний (5-6 шагов + параметры)

Варианты 18-25: сложный (7+ шагов + цветной вывод + обработка ошибок)

Варианты 1-8 (Базовый уровень)
№	Тип проекта	Требуемые шаги сборки
1	Консольный калькулятор	Проверка Python, создание venv, установка зависимостей
2	Парсер CSV-файлов	Проверка Python, установка pandas, запуск тестов
3	Веб-скрапер	Проверка Python, установка requests/bs4, проверка импортов
4	Telegram-бот	Проверка Python, установка python-telegram-bot, запуск бота
5	Генератор отчётов	Проверка Python, установка openpyxl, создание папки output
6	Анализатор логов	Проверка Python, установка pytest, создание папки logs
7	Конвертер изображений	Проверка Python, установка Pillow, проверка форматов
8	Клиент API	Проверка Python, установка requests, проверка .env файла
Пример задания (вариант 1):

python
# build.py для консольного калькулятора
# Требуется реализовать:
# 1. Проверку версии Python (>=3.8)
# 2. Создание виртуального окружения venv
# 3. Установку зависимостей из requirements.txt
# 4. Запуск тестов (если есть tests/)
# 5. Вывод отчёта
Варианты 9-17 (Средний уровень)
№	Тип проекта	Дополнительные требования
9	Веб-приложение (Flask)	+ проверка порта, создание .env, запуск сервера
10	Django-проект	+ миграции БД, сбор статики, создание суперпользователя
11	FastAPI проект	+ генерация OpenAPI схемы, проверка асинхронности
12	ML-проект	+ проверка CUDA, скачивание моделей, проверка данных
13	Библиотека Python	+ сборка wheel-пакета, проверка публикации на test PyPI
14	CLI-утилита	+ создание исполняемого файла (PyInstaller)
15	Docker-проект	+ сборка Docker образа, проверка контейнера
16	Jupyter-проект	+ проверка ядер, экспорт в HTML/PDF
17	CI/CD скрипт	+ интеграция с GitHub Actions, проверка переменных окружения
Пример задания (вариант 9):

python
# build.py для Flask-приложения
# Требуется реализовать:
# 1. Все базовые шаги
# 2. Проверку, что порт 5000 не занят
# 3. Создание файла .env из шаблона
# 4. Запуск тестов с покрытием (pytest-cov)
# 5. Запуск dev-сервера после сборки
Варианты 18-25 (Сложный уровень)
№	Тип проекта	Особенности
18	Микросервисная архитектура	Сборка нескольких сервисов, проверка зависимостей между ними
19	Мобильное приложение (Kivy)	Кроссплатформенная сборка (Android/iOS)
20	Десктоп-приложение (Tkinter/PyQt)	Создание установщика, иконок, ресурсов
21	Data Pipeline	Проверка соединений с БД, валидация данных, запуск ETL
22	Система логирования	Настройка ротации логов, проверка прав доступа
23	Плагинная архитектура	Проверка плагинов, загрузка из реестра
24	Распределённая система	Проверка соединений с узлами, синхронизация
25	Game Dev (Pygame)	Сборка ресурсов, проверка зависимостей, создание исполняемого файла
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Скрипт не работает или выполняет менее 2 шагов
3 (удовлетворительно)	Скрипт выполняет 3-4 шага, есть обработка ошибок
4 (хорошо)	Скрипт выполняет 5-6 шагов, есть параметры командной строки
5 (отлично)	Полный функционал, цветной вывод, отчёт, обработка всех ошибок
5. Шпаргалка
5.1. Основные команды для скрипта
python
# Проверка версии Python
import sys
print(f"Python {sys.version_info.major}.{sys.version_info.minor}")

# Запуск команды
import subprocess
result = subprocess.run("python --version", shell=True, capture_output=True, text=True)
print(result.stdout)

# Проверка существования файла
from pathlib import Path
if Path("requirements.txt").exists():
    print("Файл существует")

# Создание папки
Path("dist").mkdir(exist_ok=True)

# Получение аргументов командной строки
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("--clean", action="store_true")
args = parser.parse_args()

if args.clean:
    print("Очистка...")
5.2. Шаблон скрипта сборки
python
#!/usr/bin/env python3
"""build.py — Скрипт сборки проекта."""

import sys
import subprocess
from pathlib import Path

def run(cmd, desc):
    print(f"\n🔹 {desc}...")
    result = subprocess.run(cmd, shell=True)
    if result.returncode == 0:
        print(f"   ✅ {desc} завершён")
        return True
    else:
        print(f"   ❌ {desc} не удался")
        return False

def main():
    print("=" * 50)
    print("🚀 СБОРКА ПРОЕКТА")
    print("=" * 50)
    
    steps = [
        ("python --version", "Проверка Python"),
        ("python -m venv venv", "Создание venv"),
        ("source venv/bin/activate && pip install -r requirements.txt", "Установка зависимостей"),
    ]
    
    for cmd, desc in steps:
        if not run(cmd, desc):
            sys.exit(1)
    
    print("\n✅ СБОРКА ЗАВЕРШЕНА")

if __name__ == "__main__":
    main()
Карточка студента (шаблон)
text
ПЗ 2.11. НАПИСАНИЕ СКРИПТА АВТОМАТИЗАЦИИ СБОРКИ ПРОЕКТА

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Тип проекта: _________________________________

=== ТРЕБОВАНИЯ ===

Шаги сборки:
1. _________________________________
2. _________________________________
3. _________________________________
4. _________________________________
5. _________________________________

Поддерживаемые параметры:
□ --help
□ --skip-tests
□ --skip-lint
□ --clean
□ --verbose

=== ОТЧЁТ ===

Файл скрипта: build.py
Результат выполнения (скриншот): _____________
Время выполнения: _____ сек

Дата выполнения: _____________
