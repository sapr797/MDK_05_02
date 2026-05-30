# Тема 2.10. Автоматизация задач разработки с помощью скриптов Python. Сценарии для сборки, очистки, тестирования

**Цель лекции:**  
Научиться автоматизировать рутинные задачи разработки с помощью Python-скриптов: сборка проектов, очистка временных файлов, запуск тестов, генерация отчётов, деплой. Освоить создание утилит командной строки для автоматизации CI/CD процессов.

> Главная мысль: **Автоматизация — это не лень, а профессиональная эффективность. Всё, что выполняется чаще одного раза, должно быть автоматизировано.**

---

## Часть 1. Введение в автоматизацию разработки

### 1.1. Зачем автоматизировать задачи разработки

**Типичные задачи, которые можно автоматизировать:**

| Задача | Без автоматизации | С автоматизацией |
|--------|-------------------|------------------|
| Очистка временных файлов | Ручное удаление (`rm -rf`) | `python clean.py` |
| Запуск тестов | Множество команд | `python run_tests.py` |
| Сборка проекта | Длинная последовательность действий | `python build.py` |
| Генерация документации | Отдельные утилиты | `python docs.py` |
| Деплой | Множество шагов | `python deploy.py` |
| Проверка кода | Запуск линтеров по отдельности | `python lint.py` |

**Преимущества автоматизации:**
- Экономия времени (задачи выполняются за секунды вместо минут)
- Снижение человеческих ошибок (скрипт делает одно и то же каждый раз)
- Повторяемость (один скрипт для всей команды)
- Интеграция в CI/CD (скрипты можно запускать автоматически)

### 1.2. Инструменты для автоматизации

```bash
# Базовые инструменты
# 1. Python скрипты с argparse (для CLI)
# 2. Makefile (классический инструмент сборки)
# 3. Invoke (Python-альтернатива Make)
# 4. Poetry (управление зависимостями + скрипты)
# 5. Tox (тестирование в разных окружениях)
# 6. Noje (автоматизация задач)

# Установка дополнительных инструментов
pip install invoke tox poetry
Часть 2. Базовые скрипты для автоматизации
2.1. Скрипт очистки временных файлов
python
#!/usr/bin/env python3
"""
clean.py — Скрипт для очистки временных файлов и папок.

Использование:
    python clean.py           # Очистка временных файлов в текущей папке
    python clean.py --all     # Удалить все артефакты (venv, cache)
    python clean.py --dry-run # Показать, что будет удалено, без удаления
    python clean.py --verbose # Подробный вывод
"""

import os
import sys
import shutil
import argparse
from pathlib import Path
from typing import List, Tuple, Set


class ProjectCleaner:
    """Утилита для очистки проекта от временных файлов."""
    
    # Паттерны файлов для удаления
    FILE_PATTERNS: Set[str] = {
        "*.pyc",      # Скомпилированные байт-код файлы
        "*.pyo",      # Оптимизированные байт-код
        "*.so",       # Скомпилированные C-модули
        "*.egg",      # Python egg файлы
        "*.egg-info", # Информация о пакете
        "__pycache__",# Кэш Python
        ".coverage",  # Покрытие тестами
        "coverage.xml",
        ".pytest_cache",  # Кэш pytest
        ".mypy_cache",    # Кэш mypy
        ".ruff_cache",    # Кэш ruff
        ".tox",           # Виртуальные окружения tox
        "*.log",          # Лог-файлы
        "*.tmp",          # Временные файлы
        "*.bak",          # Бэкап-файлы
        "*.swp",          # Vim swap файлы
        ".DS_Store",      # macOS метаданные
        "Thumbs.db",      # Windows метаданные
    }
    
    # Папки для удаления (рекурсивно)
    DIR_PATTERNS: Set[str] = {
        "__pycache__",
        ".pytest_cache",
        ".mypy_cache",
        ".tox",
        "htmlcov",
        "dist",
        "build",
        "*.egg-info",
    }
    
    def __init__(self, root_path: Path, dry_run: bool = False, verbose: bool = False):
        self.root_path = root_path
        self.dry_run = dry_run
        self.verbose = verbose
        self.deleted_files: List[Path] = []
        self.deleted_dirs: List[Path] = []
        self.errors: List[str] = []
    
    def find_files_to_delete(self) -> List[Path]:
        """Находит файлы, соответствующие паттернам."""
        files_to_delete = []
        
        for pattern in self.FILE_PATTERNS:
            # Используем glob для поиска
            for file_path in self.root_path.glob(f"**/{pattern}"):
                if file_path.is_file():
                    files_to_delete.append(file_path)
            
            # Для специальных папок (__pycache__ и т.д.) ищем как папки
            if pattern in self.DIR_PATTERNS:
                for dir_path in self.root_path.glob(f"**/{pattern}"):
                    if dir_path.is_dir():
                        files_to_delete.append(dir_path)
        
        return files_to_delete
    
    def find_dirs_to_delete(self) -> List[Path]:
        """Находит папки для удаления."""
        dirs_to_delete = []
        
        for pattern in self.DIR_PATTERNS:
            for dir_path in self.root_path.glob(f"**/{pattern}"):
                if dir_path.is_dir():
                    dirs_to_delete.append(dir_path)
        
        return dirs_to_delete
    
    def delete_all_artifacts(self) -> None:
        """Удаляет виртуальное окружение и кэш."""
        # Виртуальные окружения
        venv_paths = list(self.root_path.glob("venv")) + \
                     list(self.root_path.glob(".venv")) + \
                     list(self.root_path.glob("env"))
        
        for venv in venv_paths:
            if venv.is_dir():
                self._delete_item(venv, is_dir=True)
        
        # Дополнительные артефакты
        site_packages = self.root_path / "site-packages"
        if site_packages.exists():
            self._delete_item(site_packages, is_dir=True)
    
    def _delete_item(self, path: Path, is_dir: bool = False) -> None:
        """Удаляет файл или папку."""
        if self.dry_run:
            if self.verbose:
                print(f"[DRY RUN] {'Папка' if is_dir else 'Файл'}: {path}")
            if is_dir:
                self.deleted_dirs.append(path)
            else:
                self.deleted_files.append(path)
            return
        
        try:
            if is_dir:
                shutil.rmtree(path)
                self.deleted_dirs.append(path)
                if self.verbose:
                    print(f"Удалена папка: {path}")
            else:
                path.unlink()
                self.deleted_files.append(path)
                if self.verbose:
                    print(f"Удалён файл: {path}")
        except Exception as e:
            error_msg = f"Ошибка при удалении {path}: {e}"
            self.errors.append(error_msg)
            print(f"  ⚠️ {error_msg}")
    
    def clean(self, all_artifacts: bool = False) -> None:
        """Выполняет очистку проекта."""
        print(f"\n🧹 Очистка проекта: {self.root_path.absolute()}")
        
        # Находим и удаляем файлы
        files = self.find_files_to_delete()
        print(f"Найдено файлов для удаления: {len(files)}")
        
        for file_path in files:
            self._delete_item(file_path, is_dir=file_path.is_dir())
        
        # Находим и удаляем папки
        dirs = self.find_dirs_to_delete()
        print(f"Найдено папок для удаления: {len(dirs)}")
        
        for dir_path in dirs:
            self._delete_item(dir_path, is_dir=True)
        
        # Полная очистка всех артефактов
        if all_artifacts:
            print("\n🧹 Полная очистка артефактов...")
            self.delete_all_artifacts()
        
        # Итоговый отчёт
        self._print_summary()
    
    def _print_summary(self) -> None:
        """Выводит отчёт об очистке."""
        print("\n" + "=" * 50)
        print("ОТЧЁТ ОБ ОЧИСТКЕ")
        print("=" * 50)
        print(f"Удалено файлов: {len(self.deleted_files)}")
        print(f"Удалено папок: {len(self.deleted_dirs)}")
        
        if self.errors:
            print(f"\n❌ Ошибок: {len(self.errors)}")
            for error in self.errors:
                print(f"  • {error}")
        
        if self.dry_run:
            print("\n⚠️ Это был пробный запуск (dry-run). Для реального удаления уберите --dry-run")
        else:
            print("\n✅ Очистка завершена!")


def main():
    parser = argparse.ArgumentParser(
        description="Очистка проекта от временных файлов и артефактов сборки",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Примеры:
  python clean.py                     # Очистка временных файлов
  python clean.py --all               # Полная очистка (включая venv)
  python clean.py --dry-run           # Показать, что будет удалено
  python clean.py --verbose --all     # Подробный вывод и полная очистка
        """
    )
    parser.add_argument(
        "--dry-run", "-n",
        action="store_true",
        help="Пробный запуск (только показать, что будет удалено)"
    )
    parser.add_argument(
        "--all", "-a",
        action="store_true",
        help="Удалить всё (включая виртуальное окружение)"
    )
    parser.add_argument(
        "--verbose", "-v",
        action="store_true",
        help="Подробный вывод"
    )
    parser.add_argument(
        "--path", "-p",
        type=str,
        default=".",
        help="Путь к проекту (по умолчанию текущая папка)"
    )
    
    args = parser.parse_args()
    
    root_path = Path(args.path).absolute()
    if not root_path.exists():
        print(f"❌ Путь не существует: {root_path}")
        sys.exit(1)
    
    cleaner = ProjectCleaner(
        root_path=root_path,
        dry_run=args.dry_run,
        verbose=args.verbose
    )
    
    cleaner.clean(all_artifacts=args.all)


if __name__ == "__main__":
    main()
2.2. Скрипт для запуска линтеров и тестов
python
#!/usr/bin/env python3
"""
check.py — Скрипт для проверки качества кода и запуска тестов.

Использование:
    python check.py           # Запуск всех проверок
    python check.py --lint    # Только линтеры
    python check.py --test    # Только тесты
    python check.py --type    # Только проверка типов (mypy)
    python check.py --fix     # Автоисправление (black, isort)
"""

import subprocess
import sys
import argparse
from pathlib import Path
from typing import List, Tuple, Dict, Any
from dataclasses import dataclass
from enum import Enum


class CheckStatus(Enum):
    """Статус проверки"""
    PASSED = "✅"
    FAILED = "❌"
    SKIPPED = "⏭️"
    ERROR = "⚠️"


@dataclass
class CheckResult:
    """Результат проверки"""
    name: str
    status: CheckStatus
    message: str = ""
    duration: float = 0.0


class CodeChecker:
    """Утилита для проверки качества кода."""
    
    def __init__(self, project_path: Path, fix: bool = False, verbose: bool = False):
        self.project_path = project_path
        self.fix = fix
        self.verbose = verbose
        self.results: List[CheckResult] = []
    
    def run_command(self, cmd: List[str], description: str) -> Tuple[bool, str]:
        """Запускает команду и возвращает результат."""
        if self.verbose:
            print(f"  Выполнение: {' '.join(cmd)}")
        
        try:
            result = subprocess.run(
                cmd,
                cwd=self.project_path,
                capture_output=True,
                text=True,
                timeout=60
            )
            
            if result.returncode == 0:
                return True, result.stdout
            else:
                return False, result.stderr or result.stdout
        
        except subprocess.TimeoutExpired:
            return False, "Timeout (60s)"
        except FileNotFoundError as e:
            return False, f"Команда не найдена: {e}"
        except Exception as e:
            return False, str(e)
    
    def run_isort(self) -> CheckResult:
        """Сортировка импортов."""
        import time
        start = time.time()
        
        if self.fix:
            cmd = ["isort", "."]
            msg = "isort (сортировка импортов)"
        else:
            cmd = ["isort", "--check", "--diff", "."]
            msg = "isort (проверка сортировки)"
        
        success, output = self.run_command(cmd, msg)
        
        return CheckResult(
            name="isort",
            status=CheckStatus.PASSED if success else CheckStatus.FAILED,
            message=output if not success else "OK",
            duration=time.time() - start
        )
    
    def run_black(self) -> CheckResult:
        """Форматирование кода."""
        import time
        start = time.time()
        
        if self.fix:
            cmd = ["black", "."]
            msg = "black (форматирование)"
        else:
            cmd = ["black", "--check", "--diff", "."]
            msg = "black (проверка форматирования)"
        
        success, output = self.run_command(cmd, msg)
        
        return CheckResult(
            name="black",
            status=CheckStatus.PASSED if success else CheckStatus.FAILED,
            message=output[:500] if not success else "OK",
            duration=time.time() - start
        )
    
    def run_flake8(self) -> CheckResult:
        """Линтер flake8 (PEP8)."""
        import time
        start = time.time()
        
        cmd = [
            "flake8", ".",
            "--max-line-length=88",
            "--extend-ignore=E203,W503",
            "--count",
            "--statistics"
        ]
        
        success, output = self.run_command(cmd, "flake8")
        
        # Подсчёт ошибок
        error_count = 0
        if output:
            try:
                lines = output.strip().split('\n')
                for line in lines:
                    if line.strip() and not line.startswith('('):
                        error_count += 1
            except:
                pass
        
        message = f"Найдено ошибок: {error_count}" if not success else "OK"
        if error_count > 0 and self.verbose:
            message += f"\n{output[:500]}"
        
        return CheckResult(
            name="flake8",
            status=CheckStatus.PASSED if success else CheckStatus.FAILED,
            message=message,
            duration=time.time() - start
        )
    
    def run_mypy(self) -> CheckResult:
        """Проверка типов mypy."""
        import time
        start = time.time()
        
        cmd = ["mypy", ".", "--ignore-missing-imports"]
        success, output = self.run_command(cmd, "mypy")
        
        # Подсчёт найденных проблем
        issue_count = len([l for l in output.split('\n') if ': error:' in l or ': note:' in l])
        
        message = f"Найдено проблем: {issue_count}" if not success else "OK"
        if issue_count > 0 and self.verbose:
            message += f"\n{output[:500]}"
        
        return CheckResult(
            name="mypy",
            status=CheckStatus.PASSED if success else CheckStatus.FAILED,
            message=message,
            duration=time.time() - start
        )
    
    def run_pytest(self) -> CheckResult:
        """Запуск тестов pytest."""
        import time
        start = time.time()
        
        cmd = [
            "pytest",
            "-v",
            "--tb=short",
            "--disable-warnings"
        ]
        
        success, output = self.run_command(cmd, "pytest")
        
        # Извлекаем статистику
        passed = 0
        failed = 0
        skipped = 0
        
        for line in output.split('\n'):
            if 'passed' in line and 'failed' in line:
                parts = line.split(',')
                for part in parts:
                    if 'passed' in part:
                        passed = int(part.strip().split()[0])
                    elif 'failed' in part:
                        failed = int(part.strip().split()[0])
                    elif 'skipped' in part:
                        skipped = int(part.strip().split()[0])
                break
        
        message = f"✅ {passed} / ❌ {failed} / ⏭️ {skipped}"
        if failed > 0 and self.verbose:
            # Показываем первые 3 ошибки
            error_lines = [l for l in output.split('\n') if 'FAILED' in l or 'ERROR' in l]
            if error_lines:
                message += f"\nОшибки:\n" + '\n'.join(error_lines[:3])
        
        return CheckResult(
            name="pytest",
            status=CheckStatus.PASSED if success else CheckStatus.FAILED,
            message=message,
            duration=time.time() - start
        )
    
    def run_pylint(self) -> CheckResult:
        """Запуск pylint для оценки качества."""
        import time
        import re
        start = time.time()
        
        cmd = ["pylint", ".", "--exit-zero", "--output-format=text"]
        success, output = self.run_command(cmd, "pylint")
        
        # Извлекаем оценку
        score_match = re.search(r"Your code has been rated at ([\d\.]+)/10", output)
        score = score_match.group(1) if score_match else "N/A"
        
        message = f"Оценка: {score}/10"
        
        return CheckResult(
            name="pylint",
            status=CheckStatus.PASSED,
            message=message,
            duration=time.time() - start
        )
    
    def run_all_checks(self, skip_tests: bool = False) -> None:
        """Запуск всех проверок."""
        print("\n" + "=" * 60)
        print(f"🔍 ПРОВЕРКА КАЧЕСТВА КОДА: {self.project_path}")
        print("=" * 60)
        
        # Линтеры
        print("\n📋 Линтеры и форматтеры:")
        self.results.append(self.run_isort())
        self.results.append(self.run_black())
        self.results.append(self.run_flake8())
        self.results.append(self.run_pylint())
        
        # Проверка типов
        print("\n🔢 Проверка типов:")
        self.results.append(self.run_mypy())
        
        # Тесты
        if not skip_tests:
            print("\n🧪 Тесты:")
            self.results.append(self.run_pytest())
        else:
            self.results.append(CheckResult("pytest", CheckStatus.SKIPPED, "Пропущено"))
        
        # Итоговый отчёт
        self._print_summary()
    
    def _print_summary(self) -> None:
        """Выводит итоговый отчёт."""
        print("\n" + "=" * 60)
        print("РЕЗУЛЬТАТЫ ПРОВЕРОК")
        print("=" * 60)
        
        all_passed = True
        
        for result in self.results:
            status_icon = result.status.value
            print(f"{status_icon} {result.name:10} | {result.message[:50]:50} | {result.duration:.2f}s")
            if result.status == CheckStatus.FAILED:
                all_passed = False
        
        total_time = sum(r.duration for r in self.results)
        print("-" * 60)
        print(f"Общее время: {total_time:.2f} сек")
        
        if all_passed:
            print("\n🎉 ВСЕ ПРОВЕРКИ ПРОЙДЕНЫ!")
        else:
            print("\n⚠️ Некоторые проверки не пройдены. Исправьте ошибки.")


def main():
    parser = argparse.ArgumentParser(
        description="Запуск проверок качества кода",
        formatter_class=argparse.RawDescriptionHelpFormatter
    )
    parser.add_argument(
        "--lint", "-l",
        action="store_true",
        help="Только линтеры (без тестов)"
    )
    parser.add_argument(
        "--test", "-t",
        action="store_true",
        help="Только тесты"
    )
    parser.add_argument(
        "--type", "-y",
        action="store_true",
        help="Только проверка типов"
    )
    parser.add_argument(
        "--fix", "-f",
        action="store_true",
        help="Автоисправление (isort, black)"
    )
    parser.add_argument(
        "--verbose", "-v",
        action="store_true",
        help="Подробный вывод"
    )
    parser.add_argument(
        "--path", "-p",
        type=str,
        default=".",
        help="Путь к проекту (по умолчанию текущая папка)"
    )
    
    args = parser.parse_args()
    project_path = Path(args.path).absolute()
    
    checker = CodeChecker(
        project_path=project_path,
        fix=args.fix,
        verbose=args.verbose
    )
    
    # Если указаны флаги, запускаем только выбранные проверки
    if args.lint:
        checker.results.append(checker.run_isort())
        checker.results.append(checker.run_black())
        checker.results.append(checker.run_flake8())
        checker.results.append(checker.run_pylint())
        checker._print_summary()
    elif args.test:
        checker.results.append(checker.run_pytest())
        checker._print_summary()
    elif args.type:
        checker.results.append(checker.run_mypy())
        checker._print_summary()
    else:
        # Запускаем все проверки
        checker.run_all_checks()


if __name__ == "__main__":
    main()
Часть 3. Скрипты сборки и деплоя
3.1. Скрипт сборки проекта
python
#!/usr/bin/env python3
"""
build.py — Скрипт для сборки Python-пакета.

Использование:
    python build.py           # Сборка проекта
    python build.py --clean   # Очистка перед сборкой
    python build.py --upload  # Загрузка на PyPI
"""

import subprocess
import sys
import argparse
import shutil
from pathlib import Path
from typing import List, Tuple
from datetime import datetime


class ProjectBuilder:
    """Утилита для сборки Python-проектов."""
    
    def __init__(self, project_path: Path, clean: bool = False):
        self.project_path = project_path
        self.clean = clean
        self.steps: List[Tuple[str, bool]] = []
    
    def run_command(self, cmd: List[str], description: str) -> bool:
        """Запускает команду и возвращает успех."""
        print(f"\n📦 {description}...")
        print(f"   Команда: {' '.join(cmd)}")
        
        try:
            result = subprocess.run(
                cmd,
                cwd=self.project_path,
                text=True,
                capture_output=True
            )
            
            if result.returncode == 0:
                print(f"   ✅ {description} завершён")
                if result.stdout:
                    print(result.stdout)
                return True
            else:
                print(f"   ❌ {description} не удался")
                print(result.stderr)
                return False
        
        except Exception as e:
            print(f"   ❌ Ошибка: {e}")
            return False
    
    def clean_build_artifacts(self) -> bool:
        """Очистка артефактов сборки."""
        print("\n🧹 Очистка артефактов сборки...")
        
        dirs_to_remove = ["dist", "build", "*.egg-info"]
        success = True
        
        for pattern in dirs_to_remove:
            for path in self.project_path.glob(f"**/{pattern}"):
                if path.is_dir():
                    print(f"   Удаление: {path}")
                    try:
                        shutil.rmtree(path)
                    except Exception as e:
                        print(f"   ⚠️ Ошибка удаления {path}: {e}")
                        success = False
        
        # Удаляем .pyc файлы
        for pyc in self.project_path.glob("**/*.pyc"):
            pyc.unlink()
        
        return success
    
    def update_version(self, version_type: str = "patch") -> bool:
        """Обновление версии проекта (упрощённо)."""
        version_file = self.project_path / "version.txt"
        
        if not version_file.exists():
            print("   version.txt не найден, пропускаем")
            return True
        
        current = version_file.read_text().strip()
        parts = current.split('.')
        
        if version_type == "major":
            parts[0] = str(int(parts[0]) + 1)
            parts[1] = "0"
            parts[2] = "0"
        elif version_type == "minor":
            parts[1] = str(int(parts[1]) + 1)
            parts[2] = "0"
        else:  # patch
            parts[2] = str(int(parts[2]) + 1)
        
        new_version = '.'.join(parts)
        version_file.write_text(new_version)
        print(f"   Версия обновлена: {current} → {new_version}")
        
        return True
    
    def install_dependencies(self) -> bool:
        """Установка зависимостей."""
        req_file = self.project_path / "requirements.txt"
        if not req_file.exists():
            print("   requirements.txt не найден")
            return True
        
        return self.run_command(
            [sys.executable, "-m", "pip", "install", "-r", "requirements.txt"],
            "Установка зависимостей"
        )
    
    def build_package(self) -> bool:
        """Сборка пакета."""
        # Проверка наличия setup.py или pyproject.toml
        has_setup = (self.project_path / "setup.py").exists()
        has_pyproject = (self.project_path / "pyproject.toml").exists()
        
        if has_setup:
            return self.run_command(
                [sys.executable, "setup.py", "sdist", "bdist_wheel"],
                "Сборка пакета (setup.py)"
            )
        elif has_pyproject:
            return self.run_command(
                [sys.executable, "-m", "build"],
                "Сборка пакета (pyproject.toml)"
            )
        else:
            print("   Не найден setup.py или pyproject.toml")
            return False
    
    def check_package(self) -> bool:
        """Проверка пакета с помощью twine."""
        return self.run_command(
            ["twine", "check", "dist/*"],
            "Проверка пакета (twine)"
        )
    
    def upload_package(self, repository: str = "testpypi") -> bool:
        """Загрузка пакета на PyPI."""
        if repository == "pypi":
            return self.run_command(
                ["twine", "upload", "dist/*"],
                "Загрузка на PyPI"
            )
        else:
            return self.run_command(
                ["twine", "upload", "--repository-url", "https://test.pypi.org/legacy/", "dist/*"],
                "Загрузка на Test PyPI"
            )
    
    def build(self, upload: bool = False) -> bool:
        """Выполняет полную сборку проекта."""
        print("\n" + "=" * 60)
        print(f"🏗️ СБОРКА ПРОЕКТА: {self.project_path.name}")
        print("=" * 60)
        
        if self.clean:
            if not self.clean_build_artifacts():
                print("\n❌ Очистка не удалась")
                return False
        
        # Шаги сборки
        steps = [
            ("Установка зависимостей", self.install_dependencies),
            ("Сборка пакета", self.build_package),
            ("Проверка пакета", self.check_package),
        ]
        
        if upload:
            steps.append(("Загрузка на PyPI", lambda: self.upload_package("testpypi")))
        
        for name, step in steps:
            if not step():
                print(f"\n❌ Сборка прервана на шаге: {name}")
                return False
        
        print("\n" + "=" * 60)
        print("🎉 СБОРКА УСПЕШНО ЗАВЕРШЕНА!")
        print("=" * 60)
        
        # Показываем артефакты
        dist_dir = self.project_path / "dist"
        if dist_dir.exists():
            print("\n📦 Артефакты сборки:")
            for artifact in dist_dir.iterdir():
                size = artifact.stat().st_size / 1024
                print(f"   • {artifact.name} ({size:.1f} KB)")
        
        return True


def main():
    parser = argparse.ArgumentParser(description="Сборка Python-проекта")
    parser.add_argument("--clean", "-c", action="store_true", help="Очистка перед сборкой")
    parser.add_argument("--upload", "-u", action="store_true", help="Загрузить на PyPI после сборки")
    parser.add_argument("--version", "-v", type=str, default="patch", choices=["patch", "minor", "major"])
    parser.add_argument("--path", "-p", type=str, default=".", help="Путь к проекту")
    
    args = parser.parse_args()
    
    builder = ProjectBuilder(
        project_path=Path(args.path).absolute(),
        clean=args.clean
    )
    
    success = builder.build(upload=args.upload)
    sys.exit(0 if success else 1)


if __name__ == "__main__":
    main()
Часть 4. Скрипты для CI/CD
4.1. Скрипт для GitHub Actions / GitLab CI
python
#!/usr/bin/env python3
"""
ci.py — Скрипт для CI/CD пайплайна.

Поддерживает:
    pre-commit хуки
    Проверка в pull request
    Автоматический деплой
"""

import os
import sys
import json
import argparse
import subprocess
from pathlib import Path
from typing import Dict, Any, List, Optional
from datetime import datetime
from enum import Enum


class CIContext(Enum):
    """Контекст выполнения CI"""
    LOCAL = "local"
    GITHUB_ACTIONS = "github_actions"
    GITLAB_CI = "gitlab_ci"
    JENKINS = "jenkins"


class CIPipeline:
    """CI/CD пайплайн."""
    
    def __init__(self):
        self.context = self._detect_context()
        self.start_time = datetime.now()
        self.results: Dict[str, bool] = {}
    
    def _detect_context(self) -> CIContext:
        """Определяет окружение CI/CD."""
        if os.environ.get("GITHUB_ACTIONS") == "true":
            return CIContext.GITHUB_ACTIONS
        elif os.environ.get("GITLAB_CI") == "true":
            return CIContext.GITLAB_CI
        elif os.environ.get("JENKINS_URL"):
            return CIContext.JENKINS
        else:
            return CIContext.LOCAL
    
    def get_pr_number(self) -> Optional[int]:
        """Получает номер Pull Request."""
        if self.context == CIContext.GITHUB_ACTIONS:
            # GitHub Actions: событие PR
            event_path = os.environ.get("GITHUB_EVENT_PATH")
            if event_path and Path(event_path).exists():
                with open(event_path) as f:
                    event = json.load(f)
                    return event.get("pull_request", {}).get("number")
        return None
    
    def set_github_output(self, name: str, value: str) -> None:
        """Устанавливает output для GitHub Actions."""
        if self.context == CIContext.GITHUB_ACTIONS:
            github_output = os.environ.get("GITHUB_OUTPUT")
            if github_output:
                with open(github_output, "a") as f:
                    f.write(f"{name}={value}\n")
    
    def set_github_annotation(self, level: str, message: str, file: str = None, line: int = None) -> None:
        """Создаёт аннотацию в GitHub Actions."""
        if self.context == CIContext.GITHUB_ACTIONS:
            annotation = f"::{level} file={file},line={line}::{message}" if file else f"::{level}::{message}"
            print(annotation, file=sys.stderr)
    
    def run_step(self, name: str, command: List[str]) -> bool:
        """Запускает шаг пайплайна."""
        print(f"\n🔹 {name}")
        print(f"   Команда: {' '.join(command)}")
        
        try:
            result = subprocess.run(
                command,
                capture_output=True,
                text=True,
                check=False
            )
            
            if result.returncode == 0:
                print(f"   ✅ Успешно")
                if result.stdout:
                    for line in result.stdout.split('\n')[:5]:
                        print(f"      {line}")
                self.results[name] = True
                return True
            else:
                print(f"   ❌ Ошибка (код {result.returncode})")
                if result.stderr:
                    for line in result.stderr.split('\n')[:10]:
                        print(f"      {line}")
                        self.set_github_annotation("error", line)
                self.results[name] = False
                return False
        
        except Exception as e:
            print
               print(f"   ❌ Исключение: {e}")
            self.results[name] = False
            return False
    
    def run_pre_commit_hooks(self) -> bool:
        """Запуск pre-commit хуков."""
        return self.run_step(
            "Pre-commit хуки",
            ["pre-commit", "run", "--all-files", "--show-diff-on-failure"]
        )
    
    def run_tests_with_coverage(self) -> bool:
        """Запуск тестов с покрытием."""
        success = self.run_step(
            "Запуск тестов с покрытием",
            ["pytest", "--cov=src", "--cov-report=xml", "--cov-report=term", "--cov-fail-under=80"]
        )
        
        # Загружаем отчёт о покрытии для PR
        if self.context == CIContext.GITHUB_ACTIONS:
            coverage_file = Path("coverage.xml")
            if coverage_file.exists():
                self.set_github_output("coverage_file", "coverage.xml")
        
        return success
    
    def run_type_checks(self) -> bool:
        """Проверка типов."""
        return self.run_step(
            "Проверка типов (mypy)",
            ["mypy", "src", "--ignore-missing-imports", "--strict"]
        )
    
    def run_linters(self) -> bool:
        """Запуск всех линтеров."""
        linters = [
            ("Flake8 (PEP8)", ["flake8", "src", "--max-line-length=88"]),
            ("Black (check)", ["black", "--check", "src"]),
            ("isort (check)", ["isort", "--check-only", "src"]),
            ("Pylint", ["pylint", "src", "--fail-under=8.0"]),
        ]
        
        all_passed = True
        for name, cmd in linters:
            if not self.run_step(name, cmd):
                all_passed = False
        
        return all_passed
    
    def run_security_checks(self) -> bool:
        """Проверка безопасности."""
        all_passed = True
        
        # Bandit (безопасность)
        if not self.run_step("Bandit (безопасность)", ["bandit", "-r", "src", "-ll"]):
            all_passed = False
        
        # Safety (уязвимости в зависимостях)
        if not self.run_step("Safety (уязвимости)", ["safety", "check", "--full-report"]):
            all_passed = False
        
        return all_passed
    
    def build_docker_image(self, tag: str = "latest") -> bool:
        """Сборка Docker образа."""
        return self.run_step(
            "Сборка Docker образа",
            ["docker", "build", "-t", f"myapp:{tag}", "."]
        )
    
    def run_full_checks(self) -> bool:
        """Запуск всех проверок."""
        print("\n" + "=" * 60)
        print(f"🚀 ЗАПУСК CI/CD ПАЙПЛАЙНА")
        print(f"   Контекст: {self.context.value}")
        print(f"   Время начала: {self.start_time.isoformat()}")
        print("=" * 60)
        
        # Определяем, запускать ли все проверки
        is_pr = self.get_pr_number() is not None
        is_main_branch = os.environ.get("GITHUB_REF") == "refs/heads/main"
        
        print(f"\n📋 Информация о запуске:")
        print(f"   Pull Request: {'Да' if is_pr else 'Нет'}")
        print(f"   Main branch: {'Да' if is_main_branch else 'Нет'}")
        
        # Шаги пайплайна
        steps = [
            ("Линтеры", self.run_linters),
            ("Проверка типов", self.run_type_checks),
            ("Проверка безопасности", self.run_security_checks),
            ("Тесты", self.run_tests_with_coverage),
        ]
        
        all_passed = True
        for name, step in steps:
            if not step():
                all_passed = False
                if self.context != CIContext.LOCAL:
                    # В CI останавливаемся на первой ошибке
                    break
        
        # Сборка Docker только для main ветки
        if is_main_branch and all_passed:
            self.build_docker_image()
        
        # Отчёт
        self._print_summary()
        
        return all_passed
    
    def _print_summary(self) -> None:
        """Выводит итоговый отчёт."""
        print("\n" + "=" * 60)
        print("РЕЗУЛЬТАТЫ CI/CD ПАЙПЛАЙНА")
        print("=" * 60)
        
        for name, success in self.results.items():
            icon = "✅" if success else "❌"
            print(f"{icon} {name}")
        
        duration = (datetime.now() - self.start_time).total_seconds()
        print(f"\n⏱️ Общее время: {duration:.1f} сек")
        
        all_success = all(self.results.values())
        if all_success:
            print("\n🎉 ПАЙПЛАЙН ВЫПОЛНЕН УСПЕШНО!")
        else:
            print("\n⚠️ НЕКОТОРЫЕ ШАГИ НЕ ПРОЙДЕНЫ!")
            sys.exit(1)


def main():
    parser = argparse.ArgumentParser(description="Запуск CI/CD пайплайна")
    parser.add_argument("--step", choices=["all", "lint", "test", "type", "security"], default="all")
    
    args = parser.parse_args()
    
    pipeline = CIPipeline()
    
    if args.step == "all":
        pipeline.run_full_checks()
    elif args.step == "lint":
        pipeline.run_linters()
    elif args.step == "test":
        pipeline.run_tests_with_coverage()
    elif args.step == "type":
        pipeline.run_type_checks()
    elif args.step == "security":
        pipeline.run_security_checks()


if __name__ == "__main__":
    main()
Часть 5. Универсальный Makefile на Python
5.1. Альтернатива Makefile на Python (Invoke)
python
# tasks.py — задачи для автоматизации
"""
Управление проектом через Invoke.

Установка:
    pip install invoke

Использование:
    invoke --list                    # Список всех задач
    invoke clean                     # Очистка проекта
    invoke test                      # Запуск тестов
    invoke check                     # Проверка качества кода
    invoke build                     # Сборка пакета
    invoke deploy --env=prod         # Деплой
"""

from invoke import task, Collection
import sys
from pathlib import Path


@task
def clean(c, all=False):
    """Очистка временных файлов."""
    print("🧹 Очистка проекта...")
    
    # Удаляем .pyc файлы
    c.run("find . -type f -name '*.pyc' -delete", warn=True)
    c.run("find . -type d -name '__pycache__' -exec rm -rf {} +", warn=True)
    
    if all:
        # Удаляем venv и артефакты сборки
        c.run("rm -rf venv .venv dist build *.egg-info", warn=True)
        print("   Полная очистка завершена")
    else:
        print("   Базовая очистка завершена")


@task
def install(c, dev=False):
    """Установка зависимостей."""
    print("📦 Установка зависимостей...")
    
    if dev:
        c.run("pip install -r requirements-dev.txt")
        c.run("pre-commit install")
    else:
        c.run("pip install -r requirements.txt")
    
    print("✅ Готово")


@task(pre=[clean])
def lint(c):
    """Запуск линтеров."""
    print("🔍 Запуск линтеров...")
    
    # isort
    c.run("isort --check-only src tests", warn=True)
    
    # black
    c.run("black --check src tests", warn=True)
    
    # flake8
    c.run("flake8 src tests --max-line-length=88", warn=True)
    
    # pylint
    c.run("pylint src --fail-under=8.0", warn=True)
    
    print("✅ Линтеры выполнены")


@task
def type_check(c):
    """Проверка типов."""
    print("🔢 Проверка типов...")
    c.run("mypy src --ignore-missing-imports")
    print("✅ Проверка типов выполнена")


@task
def test(c, coverage=False):
    """Запуск тестов."""
    print("🧪 Запуск тестов...")
    
    if coverage:
        c.run("pytest --cov=src --cov-report=term --cov-report=html")
        print("   Отчёт о покрытии: htmlcov/index.html")
    else:
        c.run("pytest -v")
    
    print("✅ Тесты выполнены")


@task
def check(c):
    """Полная проверка качества кода."""
    print("\n" + "=" * 50)
    print("ПОЛНАЯ ПРОВЕРКА КАЧЕСТВА")
    print("=" * 50)
    
    lint(c)
    type_check(c)
    test(c, coverage=True)
    
    print("\n🎉 ВСЕ ПРОВЕРКИ ПРОЙДЕНЫ!")


@task(pre=[clean])
def build(c):
    """Сборка пакета."""
    print("🏗️ Сборка пакета...")
    
    # Проверка наличия setup.py
    if Path("setup.py").exists():
        c.run("python setup.py sdist bdist_wheel")
    else:
        c.run("python -m build")
    
    print("✅ Сборка завершена")


@task
def publish(c, test=True):
    """Публикация на PyPI."""
    print("📤 Публикация пакета...")
    
    if test:
        c.run("twine upload --repository-url https://test.pypi.org/legacy/ dist/*")
    else:
        c.run("twine upload dist/*")
    
    print("✅ Публикация завершена")


@task
def docs(c, serve=False):
    """Генерация документации."""
    print("📚 Генерация документации...")
    
    # Проверяем наличие sphinx
    c.run("sphinx-build -b html docs/source docs/build")
    
    if serve:
        print("   Запуск локального сервера...")
        c.run("python -m http.server --directory docs/build 8000")
    
    print("✅ Документация сгенерирована")


@task
def deploy(c, env="staging"):
    """Деплой приложения."""
    print(f"🚀 Деплой на {env}...")
    
    if env == "staging":
        c.run("git push origin staging")
        c.run("ssh deploy@staging-server 'cd /app && git pull && systemctl restart app'")
    elif env == "prod":
        # Подтверждение
        confirm = input("Вы уверены, что хотите деплоить на PRODUCTION? [y/N] ")
        if confirm.lower() == 'y':
            c.run("git push origin main")
            c.run("ssh deploy@prod-server 'cd /app && git pull && systemctl restart app'")
        else:
            print("   Деплой отменён")
    
    print("✅ Деплой завершён")


@task
def docker_build(c, tag="latest"):
    """Сборка Docker образа."""
    print("🐳 Сборка Docker образа...")
    c.run(f"docker build -t myapp:{tag} .")
    print("✅ Образ собран")


@task
def docker_run(c, port=8000):
    """Запуск Docker контейнера."""
    print("🐳 Запуск контейнера...")
    c.run(f"docker run -p {port}:8000 myapp:latest")
    print("✅ Контейнер запущен")


@task
def help(c):
    """Показать справку."""
    print("""
Доступные задачи:

    clean           - Очистка временных файлов
    install         - Установка зависимостей
    lint            - Запуск линтеров
    type-check      - Проверка типов
    test            - Запуск тестов
    check           - Полная проверка качества
    build           - Сборка пакета
    publish         - Публикация на PyPI
    docs            - Генерация документации
    deploy          - Деплой приложения
    docker-build    - Сборка Docker образа
    docker-run      - Запуск Docker контейнера
    help            - Эта справка

Примеры:
    invoke check
    invoke test --coverage
    invoke deploy --env=prod
    invoke docker-build --tag=v1.0.0
""")


# Настройка пространства имён
ns = Collection()
ns.add_task(clean)
ns.add_task(install)
ns.add_task(lint)
ns.add_task(type_check)
ns.add_task(test)
ns.add_task(check)
ns.add_task(build)
ns.add_task(publish)
ns.add_task(docs)
ns.add_task(deploy)
ns.add_task(docker_build)
ns.add_task(docker_run)
ns.add_task(help)
Шпаргалка по автоматизации
bash
# === БЫСТРЫЙ СТАРТ ===

# 1. Создание проекта
python -m venv venv
source venv/bin/activate  # или venv\Scripts\activate
pip install -r requirements-dev.txt
pre-commit install

# 2. Очистка
python clean.py
python clean.py --all --dry-run

# 3. Проверка качества
python check.py           # Все проверки
python check.py --lint    # Только линтеры
python check.py --test    # Только тесты
python check.py --fix     # Автоисправление

# 4. Сборка
python build.py           # Сборка
python build.py --clean   # Очистка + сборка
python build.py --upload  # Загрузка на PyPI

# 5. Через invoke
invoke --list
invoke check
invoke test --coverage
invoke deploy --env=staging
Контрольные вопросы
Какие задачи разработки можно автоматизировать с помощью Python-скриптов?

Зачем нужен аргумент --dry-run в скриптах очистки?

Как организовать запуск нескольких линтеров одной командой?

В чём отличие между скриптами для локального использования и CI/CD?

Какие инструменты Python существуют для создания Makefile-подобных задач?

Как обрабатывать ошибки в скриптах автоматизации?

Зачем нужен argparse при создании скриптов командной строки?

Как передать параметры из CI/CD окружения в Python-скрипт?

Практическое задание
Задание: Разработайте скрипт автоматизации для своего учебного проекта, который должен:

Очищать временные файлы и кэш

Запускать линтеры (pylint, flake8)

Запускать тесты (pytest)

Генерировать отчёт о выполнении

Поддерживать параметры командной строки (--clean, --test, --all)

Итог лекции
Вы сегодня:

Освоили написание скриптов для очистки проектов

Научились автоматизировать запуск линтеров и тестов

Изучили скрипты сборки и деплоя

Познакомились с CI/CD скриптами

Узнали об альтернативах Makefile (Invoke)

Теперь вы можете автоматизировать практически любую рутинную задачу разработки!

 
