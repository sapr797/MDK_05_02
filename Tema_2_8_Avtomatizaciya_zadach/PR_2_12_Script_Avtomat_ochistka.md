# ПЗ 2.12. Написание скрипта автоматизации очистки проекта и запуска всех тестов

**Тема:** Основные конструкции языка программирования: переменные, ввод-вывод, циклы, условные операторы

**Цель работы:**  
Закрепить навыки использования переменных, ввода-вывода данных, условных конструкций и циклов на примере разработки скрипта автоматизации очистки проекта и запуска тестов. Научиться создавать утилиты для поддержания чистоты кода и проверки работоспособности.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `pytest`, `pytest-cov`, `flake8`, `pylint`

```bash
pip install pytest pytest-cov flake8 pylint
Главная мысль: Чистота проекта — залог успешной разработки. Автоматизируй очистку и тестирование, чтобы сосредоточиться на коде.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Что такое скрипт очистки и тестирования
Скрипт очистки — удаляет временные файлы, кэш, артефакты сборки, которые накапливаются в процессе разработки.

Скрипт тестирования — запускает все тесты проекта и сообщает о результатах.

Объединённый скрипт — сначала очищает проект, затем запускает тесты, обеспечивая чистоту эксперимента.

1.2. Какие файлы нужно удалять
Тип файлов	Расширения/папки	Описание
Байт-код Python	*.pyc, *.pyo	Скомпилированные файлы
Кэш Python	__pycache__/	Кэш байт-кода
Кэш тестов	.pytest_cache/, .tox/	Кэш pytest и tox
Кэш линтеров	.mypy_cache/, .ruff_cache/	Кэш mypy и ruff
Артефакты сборки	dist/, build/, *.egg-info/	Результаты сборки
Отчёты о покрытии	htmlcov/, .coverage	Отчёты pytest-cov
Лог-файлы	*.log	Логи приложения
Системные файлы	.DS_Store, Thumbs.db	Файлы ОС
1.3. Команды для запуска тестов
bash
# Запуск всех тестов
pytest

# Запуск с подробным выводом
pytest -v

# Запуск с отчётом о покрытии
pytest --cov=src --cov-report=term

# Запуск конкретного теста
pytest tests/test_main.py

# Запуск с остановкой при первой ошибке
pytest -x
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая разработку скрипта очистки и тестирования.

Техническое задание (нулевой вариант)
Разработайте скрипт clean_and_test.py, который:

Очищает проект от временных файлов (.pyc, __pycache__, .pytest_cache, .coverage, htmlcov/)

Выводит отчёт об удалённых файлах

Запускает все тесты с проверкой покрытия кода

Выводит цветной отчёт о результатах

Поддерживает параметры: --dry-run (пробный запуск), --verbose (подробный вывод), --no-tests (только очистка)

Эталонная реализация
python
#!/usr/bin/env python3
"""
clean_and_test.py — Скрипт автоматизации очистки проекта и запуска тестов.

Использование:
    python clean_and_test.py                # Очистка + тесты
    python clean_and_test.py --dry-run      # Показать, что будет удалено
    python clean_and_test.py --verbose      # Подробный вывод
    python clean_and_test.py --no-tests     # Только очистка
    python clean_and_test.py --skip-clean   # Только тесты
"""

import os
import sys
import argparse
import subprocess
from pathlib import Path
from datetime import datetime
from typing import List, Tuple


# ============================================================
# ЦВЕТА ДЛЯ ВЫВОДА
# ============================================================

class Colors:
    """Цвета для терминального вывода."""
    HEADER = '\033[95m'
    BLUE = '\033[94m'
    GREEN = '\033[92m'
    YELLOW = '\033[93m'
    RED = '\033[91m'
    END = '\033[0m'
    BOLD = '\033[1m'
    DIM = '\033[2m'


def print_color(text: str, color: str = Colors.GREEN) -> None:
    """Выводит цветной текст."""
    print(f"{color}{text}{Colors.END}")


def print_success(text: str) -> None:
    """Выводит сообщение об успехе."""
    print(f"  {Colors.GREEN}✅ {text}{Colors.END}")


def print_error(text: str) -> None:
    """Выводит сообщение об ошибке."""
    print(f"  {Colors.RED}❌ {text}{Colors.END}")


def print_warning(text: str) -> None:
    """Выводит предупреждение."""
    print(f"  {Colors.YELLOW}⚠️ {text}{Colors.END}")


def print_info(text: str) -> None:
    """Выводит информационное сообщение."""
    print(f"  {Colors.BLUE}📌 {text}{Colors.END}")


def print_step(text: str) -> None:
    """Выводит заголовок шага."""
    print(f"\n{Colors.BOLD}{Colors.BLUE}▶ {text}{Colors.END}")


# ============================================================
# ОЧИСТКА ПРОЕКТА
# ============================================================

class ProjectCleaner:
    """Класс для очистки проекта от временных файлов."""
    
    # Паттерны файлов для удаления
    FILE_PATTERNS: List[str] = [
        "*.pyc",           # Скомпилированные байт-код файлы
        "*.pyo",           # Оптимизированные байт-код
        "*.so",            # Скомпилированные C-модули
        ".coverage",       # Отчёт о покрытии
        "coverage.xml",    # Отчёт о покрытии в XML
        "*.log",           # Лог-файлы
        "*.tmp",           # Временные файлы
        "*.bak",           # Резервные копии
        "*.swp",           # Vim swap файлы
        ".DS_Store",       # macOS метаданные
        "Thumbs.db",       # Windows метаданные
    ]
    
    # Папки для удаления
    DIR_PATTERNS: List[str] = [
        "__pycache__",     # Кэш Python
        ".pytest_cache",   # Кэш pytest
        ".mypy_cache",     # Кэш mypy
        ".ruff_cache",     # Кэш ruff
        ".tox",            # Виртуальные окружения tox
        "htmlcov",         # Отчёт о покрытии в HTML
        "dist",            # Сборка для распространения
        "build",           # Временные файлы сборки
        "*.egg-info",      # Информация о пакете
        ".venv_cache",     # Кэш виртуального окружения
    ]
    
    def __init__(self, root_path: Path, dry_run: bool = False, verbose: bool = False):
        self.root_path = root_path
        self.dry_run = dry_run
        self.verbose = verbose
        self.deleted_files: List[Path] = []
        self.deleted_dirs: List[Path] = []
        self.errors: List[str] = []
    
    def find_files_to_delete(self) -> List[Path]:
        """Находит файлы, соответствующие паттернам."""
        files = []
        for pattern in self.FILE_PATTERNS:
            for file_path in self.root_path.glob(f"**/{pattern}"):
                if file_path.is_file():
                    files.append(file_path)
                    if self.verbose:
                        print_info(f"Найден файл: {file_path.relative_to(self.root_path)}")
        return files
    
    def find_dirs_to_delete(self) -> List[Path]:
        """Находит папки для удаления."""
        dirs = []
        for pattern in self.DIR_PATTERNS:
            for dir_path in self.root_path.glob(f"**/{pattern}"):
                if dir_path.is_dir():
                    dirs.append(dir_path)
                    if self.verbose:
                        print_info(f"Найдена папка: {dir_path.relative_to(self.root_path)}")
        return dirs
    
    def _delete_file(self, file_path: Path) -> bool:
        """Удаляет файл."""
        if self.dry_run:
            self.deleted_files.append(file_path)
            return True
        
        try:
            file_path.unlink()
            self.deleted_files.append(file_path)
            return True
        except Exception as e:
            self.errors.append(f"Ошибка удаления {file_path}: {e}")
            return False
    
    def _delete_dir(self, dir_path: Path) -> bool:
        """Удаляет папку."""
        if self.dry_run:
            self.deleted_dirs.append(dir_path)
            return True
        
        try:
            import shutil
            shutil.rmtree(dir_path)
            self.deleted_dirs.append(dir_path)
            return True
        except Exception as e:
            self.errors.append(f"Ошибка удаления {dir_path}: {e}")
            return False
    
    def clean(self) -> None:
        """Выполняет очистку проекта."""
        print_step("ОЧИСТКА ПРОЕКТА")
        
        if self.dry_run:
            print_warning("РЕЖИМ ПРОБНОГО ЗАПУСКА (файлы не будут удалены)")
        
        # Удаление файлов
        files = self.find_files_to_delete()
        for file_path in files:
            if self._delete_file(file_path):
                if self.verbose:
                    print_success(f"Удалён файл: {file_path.relative_to(self.root_path)}")
        
        # Удаление папок
        dirs = self.find_dirs_to_delete()
        for dir_path in dirs:
            if self._delete_dir(dir_path):
                if self.verbose:
                    print_success(f"Удалена папка: {dir_path.relative_to(self.root_path)}")
    
    def print_report(self) -> None:
        """Выводит отчёт об очистке."""
        print("\n" + "-" * 40)
        print("📊 ОТЧЁТ ОБ ОЧИСТКЕ")
        print("-" * 40)
        
        if self.deleted_files:
            print(f"   Удалено файлов: {len(self.deleted_files)}")
        else:
            print(f"   Удалено файлов: 0")
        
        if self.deleted_dirs:
            print(f"   Удалено папок: {len(self.deleted_dirs)}")
        else:
            print(f"   Удалено папок: 0")
        
        if self.errors:
            print_error(f"   Ошибок: {len(self.errors)}")
            for error in self.errors:
                print(f"     • {error}")
        
        if self.dry_run:
            print_warning("   Это был пробный запуск (файлы не удалены)")


# ============================================================
# ЗАПУСК ТЕСТОВ
# ============================================================

class TestRunner:
    """Класс для запуска тестов."""
    
    def __init__(self, project_path: Path, verbose: bool = False):
        self.project_path = project_path
        self.verbose = verbose
        self.passed = 0
        self.failed = 0
        self.skipped = 0
        self.total = 0
        self.coverage_percent = 0.0
    
    def run_tests(self, with_coverage: bool = True) -> bool:
        """Запускает тесты pytest."""
        print_step("ЗАПУСК ТЕСТОВ")
        
        if with_coverage:
            cmd = [
                "pytest", "-v", "--tb=short", "--disable-warnings",
                "--cov=src", "--cov-report=term"
            ]
        else:
            cmd = ["pytest", "-v", "--tb=short", "--disable-warnings"]
        
        print_info(f"Команда: {' '.join(cmd)}")
        
        try:
            result = subprocess.run(
                cmd,
                cwd=self.project_path,
                capture_output=True,
                text=True
            )
            
            self._parse_output(result.stdout + result.stderr)
            
            if result.returncode == 0:
                print_success("Все тесты пройдены успешно!")
                return True
            else:
                print_error(f"Неудачных тестов: {self.failed}")
                return False
        
        except FileNotFoundError:
            print_error("pytest не установлен. Установите: pip install pytest")
            return False
        except Exception as e:
            print_error(f"Ошибка при запуске тестов: {e}")
            return False
    
    def _parse_output(self, output: str) -> None:
        """Парсит вывод pytest."""
        lines = output.split('\n')
        
        # Поиск строки с результатами тестов
        for line in lines:
            if 'passed' in line and 'failed' in line:
                parts = line.split(',')
                for part in parts:
                    part = part.strip()
                    if 'passed' in part:
                        self.passed = int(part.split()[0])
                    elif 'failed' in part:
                        self.failed = int(part.split()[0])
                    elif 'skipped' in part:
                        self.skipped = int(part.split()[0])
            elif 'TOTAL' in line and '%' in line:
                # Поиск процента покрытия
                import re
                match = re.search(r'(\d+)%', line)
                if match:
                    self.coverage_percent = int(match.group(1))
        
        self.total = self.passed + self.failed + self.skipped
    
    def print_report(self) -> None:
        """Выводит отчёт о тестах."""
        print("\n" + "-" * 40)
        print("📊 ОТЧЁТ О ТЕСТАХ")
        print("-" * 40)
        
        print(f"   {Colors.GREEN}✅ Пройдено: {self.passed}{Colors.END}")
        if self.failed > 0:
            print(f"   {Colors.RED}❌ Неудачно: {self.failed}{Colors.END}")
        if self.skipped > 0:
            print(f"   {Colors.YELLOW}⏭️ Пропущено: {self.skipped}{Colors.END}")
        print(f"   📊 Всего тестов: {self.total}")
        
        if self.coverage_percent > 0:
            coverage_color = Colors.GREEN if self.coverage_percent >= 80 else (Colors.YELLOW if self.coverage_percent >= 60 else Colors.RED)
            print(f"   📈 Покрытие кода: {coverage_color}{self.coverage_percent}%{Colors.END}")


# ============================================================
# ОСНОВНАЯ ФУНКЦИЯ
# ============================================================

def main():
    """Главная функция скрипта."""
    parser = argparse.ArgumentParser(
        description="Автоматизация очистки проекта и запуска тестов",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Примеры:
  python clean_and_test.py                # Очистка + тесты
  python clean_and_test.py --dry-run      # Показать, что будет удалено
  python clean_and_test.py --verbose      # Подробный вывод
  python clean_and_test.py --no-tests     # Только очистка
  python clean_and_test.py --skip-clean   # Только тесты
  python clean_and_test.py --no-coverage  # Тесты без покрытия
        """
    )
    parser.add_argument("--dry-run", "-n", action="store_true", help="Пробный запуск (без удаления)")
    parser.add_argument("--verbose", "-v", action="store_true", help="Подробный вывод")
    parser.add_argument("--no-tests", "-T", action="store_true", help="Пропустить тесты")
    parser.add_argument("--skip-clean", "-C", action="store_true", help="Пропустить очистку")
    parser.add_argument("--no-coverage", action="store_true", help="Запустить тесты без проверки покрытия")
    parser.add_argument("--path", "-p", type=str, default=".", help="Путь к проекту")
    
    args = parser.parse_args()
    
    # Определяем путь к проекту
    project_path = Path(args.path).absolute()
    start_time = datetime.now()
    
    # Вывод заголовка
    print("\n" + "=" * 60)
    print_color("🧹 АВТОМАТИЗАЦИЯ ОЧИСТКИ И ТЕСТИРОВАНИЯ", Colors.BOLD)
    print(f"   Путь: {project_path}")
    print(f"   Время: {start_time.strftime('%H:%M:%S')}")
    print("=" * 60)
    
    success = True
    
    # Очистка проекта
    if not args.skip_clean:
        cleaner = ProjectCleaner(project_path, args.dry_run, args.verbose)
        cleaner.clean()
        cleaner.print_report()
    else:
        print_warning("\n⏭️ Очистка пропущена")
    
    # Запуск тестов
    if not args.no_tests:
        runner = TestRunner(project_path, args.verbose)
        test_success = runner.run_tests(with_coverage=not args.no_coverage)
        runner.print_report()
        success = success and test_success
    else:
        print_warning("\n⏭️ Тесты пропущены")
    
    # Итоговый отчёт
    end_time = datetime.now()
    duration = (end_time - start_time).total_seconds()
    
    print("\n" + "=" * 60)
    print_color("📊 ИТОГОВЫЙ ОТЧЁТ", Colors.BOLD)
    print("=" * 60)
    print(f"   ⏱️ Время выполнения: {duration:.2f} сек")
    
    if success:
        print_color("\n🎉 ВСЕ ОПЕРАЦИИ ВЫПОЛНЕНЫ УСПЕШНО!", Colors.GREEN)
    else:
        print_color("\n⚠️ НЕКОТОРЫЕ ОПЕРАЦИИ ЗАВЕРШИЛИСЬ С ОШИБКАМИ!", Colors.RED)
        sys.exit(1)
    
    print("=" * 60 + "\n")


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
============================================================
🧹 АВТОМАТИЗАЦИЯ ОЧИСТКИ И ТЕСТИРОВАНИЯ
   Путь: /home/user/my_project
   Время: 14:30:25
============================================================

▶ ОЧИСТКА ПРОЕКТА

📌 Найден файл: src/__pycache__/main.cpython-310.pyc
📌 Найдена папка: .pytest_cache
  ✅ Удалён файл: src/__pycache__/main.cpython-310.pyc
  ✅ Удалена папка: .pytest_cache

----------------------------------------
📊 ОТЧЁТ ОБ ОЧИСТКЕ
----------------------------------------
   Удалено файлов: 3
   Удалено папок: 2

▶ ЗАПУСК ТЕСТОВ

📌 Команда: pytest -v --tb=short --disable-warnings --cov=src --cov-report=term
========= test session starts ==========
tests/test_main.py::test_add PASSED
tests/test_main.py::test_subtract PASSED
tests/test_main.py::test_multiply PASSED
tests/test_main.py::test_divide PASSED
========= 4 passed in 0.15s ===========

  ✅ Все тесты пройдены успешно!

----------------------------------------
📊 ОТЧЁТ О ТЕСТАХ
----------------------------------------
   ✅ Пройдено: 4
   📊 Всего тестов: 4
   📈 Покрытие кода: 95%

============================================================
📊 ИТОГОВЫЙ ОТЧЁТ
============================================================
   ⏱️ Время выполнения: 2.34 сек

🎉 ВСЕ ОПЕРАЦИИ ВЫПОЛНЕНЫ УСПЕШНО!
============================================================
3. 25 вариантов практической работы
Общая структура каждого варианта:
Студент разрабатывает скрипт автоматизации очистки и тестирования для заданного типа проекта.

Уровни сложности:

Варианты 1-8: базовый (очистка + базовые тесты)

Варианты 9-17: средний (расширенная очистка + покрытие кода)

Варианты 18-25: сложный (многомодульные проекты + CI/CD интеграция)

Варианты 1-8 (Базовый уровень)
№	Тип проекта	Дополнительные требования
1	Консольный калькулятор	Очистка .pyc, тесты на арифметику
2	Парсер CSV	Очистка __pycache__, тесты на чтение файлов
3	Валидатор email	Очистка .pytest_cache, тесты на валидацию
4	Конвертер валют	Очистка .coverage, тесты на конвертацию
5	Генератор паролей	Очистка *.log, тесты на длину пароля
6	Калькулятор скидок	Очистка htmlcov/, тесты на скидки
7	Таймер обратного отсчёта	Очистка .mypy_cache, тесты на время
8	Конвертер температур	Очистка dist/, тесты на формулы
Варианты 9-17 (Средний уровень)
№	Тип проекта	Дополнительные требования
9	Flask-приложение	+ очистка .env, тесты маршрутов
10	Django-проект	+ очистка миграций, тесты моделей
11	FastAPI проект	+ очистка *.db, тесты эндпоинтов
12	Telegram-бот	+ очистка логов, тесты команд
13	Библиотека Python	+ очистка dist/, тесты импортов
14	CLI-утилита	+ очистка *.exe, тесты аргументов
15	Парсер сайтов	+ очистка кэша, тесты парсинга
16	REST API клиент	+ очистка токенов, тесты запросов
17	Анализатор логов	+ очистка временных файлов, тесты анализа
Варианты 18-25 (Сложный уровень)
№	Тип проекта	Дополнительные требования
18	Микросервис	Очистка нескольких папок, запуск тестов каждого сервиса
19	ML-проект	Очистка моделей, кэша датасетов, тесты обучения
20	Десктоп-приложение	Очистка ресурсов, тесты GUI
21	ETL-пайплайн	Очистка временных БД, тесты трансформаций
22	Плагинная система	Очистка плагинов, тесты загрузки
23	Game Dev (Pygame)	Очистка ресурсов, тесты физики
24	Система мониторинга	Очистка метрик, тесты алертов
25	Асинхронный парсер	Очистка очередей, тесты async/await
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Скрипт не работает или выполняет менее 2 функций
3 (удовлетворительно)	Реализованы очистка ИЛИ тесты, 3-4 шага
4 (хорошо)	Реализованы очистка И тесты, есть параметры командной строки
5 (отлично)	Полный функционал, цветной вывод, отчёты, обработка ошибок
5. Шпаргалка
5.1. Основные команды для скрипта
python
# Удаление файла
from pathlib import Path
file_path = Path("temp.txt")
file_path.unlink()

# Удаление папки
import shutil
shutil.rmtree("__pycache__")

# Поиск файлов по шаблону
for file in Path(".").glob("**/*.pyc"):
    print(file)

# Запуск pytest
import subprocess
result = subprocess.run(["pytest", "-v"], capture_output=True, text=True)
print(result.stdout)

# Парсинг вывода
for line in result.stdout.split('\n'):
    if 'passed' in line:
        print(line)
5.2. Шаблон скрипта очистки и тестов
python
#!/usr/bin/env python3
"""clean_test.py — Очистка и тестирование."""

import sys
import subprocess
from pathlib import Path

def clean():
    """Очистка временных файлов."""
    patterns = ["*.pyc", "__pycache__", ".pytest_cache"]
    
    for pattern in patterns:
        for path in Path(".").glob(f"**/{pattern}"):
            if path.is_file():
                path.unlink()
                print(f"Удалён файл: {path}")
            elif path.is_dir():
                import shutil
                shutil.rmtree(path)
                print(f"Удалена папка: {path}")

def run_tests():
    """Запуск тестов."""
    result = subprocess.run(["pytest", "-v", "--tb=short"])
    return result.returncode == 0

def main():
    print("🧹 Очистка проекта...")
    clean()
    
    print("\n🧪 Запуск тестов...")
    if run_tests():
        print("✅ Все тесты пройдены!")
    else:
        print("❌ Некоторые тесты не пройдены!")
        sys.exit(1)

if __name__ == "__main__":
    main()
Карточка студента (шаблон)
text
ПЗ 2.12. НАПИСАНИЕ СКРИПТА АВТОМАТИЗАЦИИ ОЧИСТКИ И ТЕСТИРОВАНИЯ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Тип проекта: _________________________________

=== ОЧИСТКА ===

Удаляемые файлы: _________________________________
Удаляемые папки: _________________________________
Режим dry-run: □ Да □ Нет

=== ТЕСТИРОВАНИЕ ===

Команда запуска тестов: _________________________________
Покрытие кода: □ Да □ Нет
Минимальный порог покрытия: _____%

=== ОТЧЁТ ===

Файл скрипта: clean_and_test.py
Результат выполнения (скриншот): _____________
Время выполнения: _____ сек

Дата выполнения: _____________
