# ПЗ 2.50. Создание CLI-интерфейса для управления проектом с помощью argparse

**Тема:** Командная строка, аргументы, подкоманды, автодополнение

**Цель работы:**  
Научиться создавать интерфейс командной строки (CLI) для управления проектом с использованием модуля `argparse`, реализовывать подкоманды, обрабатывать аргументы и опции, создавать удобные и документированные CLI-утилиты.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+

> Главная мысль: **CLI — это лицо вашего инструмента. Хороший интерфейс командной строки должен быть интуитивным, документированным и удобным для автоматизации.**

---

## Содержание

1. [Теоретическая справка](#1-теоретическая-справка)
2. [Нулевой вариант (эталонный)](#2-нулевой-вариант-эталонный)
3. [25 вариантов практической работы](#3-25-вариантов-практической-работы)
4. [Критерии оценки](#4-критерии-оценки)
5. [Шпаргалка](#5-шпаргалка)

---

## 1. Теоретическая справка

### 1.1. Что такое CLI и зачем он нужен

**CLI (Command Line Interface)** — интерфейс командной строки для взаимодействия с программой.
┌─────────────────────────────────────────────────────────────────────────────┐
│ СТРУКТУРА CLI-ПРИЛОЖЕНИЯ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ mycli [ГЛОБАЛЬНЫЕ_ОПЦИИ] КОМАНДА [АРГУМЕНТЫ] [ОПЦИИ_КОМАНДЫ] │
│ │
│ Примеры: │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ mycli --help │ │
│ │ mycli user create --name Ivan --email ivan@example.com │ │
│ │ mycli task list --status pending --limit 10 │ │
│ │ mycli db migrate --version 001 │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Компоненты argparse

| Компонент | Описание | Пример |
|-----------|----------|--------|
| **Позиционные аргументы** | Обязательные, порядок важен | `mycli create task` |
| **Опциональные аргументы** | С флагами, необязательные | `--name Ivan` |
| **Флаги** | Без значения, True/False | `--verbose` |
| **Подкоманды** | Вложенные команды | `user create`, `task list` |
| **Значения по умолчанию** | Если аргумент не указан | `default=10` |
| **Валидация** | Проверка типов и диапазонов | `type=int`, `choices=[...]` |

---

## 2. Нулевой вариант (эталонный)

**Назначение:** преподаватель выполняет этот вариант на занятии, показывая создание CLI-интерфейса для управления проектом.

### Техническое задание (нулевой вариант)

> Разработайте CLI-утилиту `taskcli` для управления задачами с поддержкой:
> - Команд для работы с задачами (create, list, update, delete, complete)
> - Команд для работы с пользователями (register, login, logout)
> - Опций для фильтрации, пагинации и форматирования вывода
> - Цветного вывода и прогресс-баров
> - Конфигурационного файла для хранения настроек

### Эталонная реализация

```python
#!/usr/bin/env python3
"""
taskcli.py — CLI-интерфейс для управления задачами.
"""

import argparse
import sys
import json
import os
from pathlib import Path
from datetime import datetime
from typing import Optional, List, Dict, Any
import textwrap

# ============================================================
# КОНСТАНТЫ И КОНФИГУРАЦИЯ
# ============================================================

CONFIG_DIR = Path.home() / ".taskcli"
CONFIG_FILE = CONFIG_DIR / "config.json"
TOKEN_FILE = CONFIG_DIR / "token.json"
API_URL = "http://localhost:8000"

# Цвета для вывода
class Colors:
    HEADER = '\033[95m'
    BLUE = '\033[94m'
    GREEN = '\033[92m'
    YELLOW = '\033[93m'
    RED = '\033[91m'
    END = '\033[0m'
    BOLD = '\033[1m'
    DIM = '\033[2m'

def print_color(text: str, color: str = Colors.GREEN):
    print(f"{color}{text}{Colors.END}")


# ============================================================
# КОНФИГУРАЦИЯ
# ============================================================

def ensure_config_dir():
    """Создание директории конфигурации."""
    CONFIG_DIR.mkdir(exist_ok=True, parents=True)


def load_config() -> Dict:
    """Загрузка конфигурации."""
    ensure_config_dir()
    if CONFIG_FILE.exists():
        with open(CONFIG_FILE, 'r') as f:
            return json.load(f)
    return {"api_url": API_URL, "default_limit": 10, "color": True}


def save_config(config: Dict):
    """Сохранение конфигурации."""
    ensure_config_dir()
    with open(CONFIG_FILE, 'w') as f:
        json.dump(config, f, indent=2)


def save_token(token: str):
    """Сохранение токена авторизации."""
    ensure_config_dir()
    with open(TOKEN_FILE, 'w') as f:
        json.dump({"token": token, "saved_at": datetime.now().isoformat()}, f)


def load_token() -> Optional[str]:
    """Загрузка токена авторизации."""
    if TOKEN_FILE.exists():
        with open(TOKEN_FILE, 'r') as f:
            data = json.load(f)
            return data.get("token")
    return None


def clear_token():
    """Удаление токена."""
    if TOKEN_FILE.exists():
        TOKEN_FILE.unlink()


# ============================================================
# HTTP-КЛИЕНТ
# ============================================================

import requests


class APIClient:
    """Клиент для работы с API."""
    
    def __init__(self, base_url: str = None):
        self.base_url = base_url or load_config().get("api_url", API_URL)
        self.token = load_token()
    
    def _get_headers(self) -> Dict:
        headers = {"Content-Type": "application/json"}
        if self.token:
            headers["Authorization"] = f"Bearer {self.token}"
        return headers
    
    def get(self, endpoint: str, params: Dict = None) -> Dict:
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        response = requests.get(url, headers=self._get_headers(), params=params)
        response.raise_for_status()
        return response.json()
    
    def post(self, endpoint: str, data: Dict = None) -> Dict:
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        response = requests.post(url, headers=self._get_headers(), json=data)
        response.raise_for_status()
        return response.json()
    
    def put(self, endpoint: str, data: Dict = None) -> Dict:
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        response = requests.put(url, headers=self._get_headers(), json=data)
        response.raise_for_status()
        return response.json()
    
    def delete(self, endpoint: str) -> bool:
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        response = requests.delete(url, headers=self._get_headers())
        return response.status_code == 204


# ============================================================
# ФОРМАТТЕРЫ ВЫВОДА
# ============================================================

def format_task(task: Dict, index: int = None) -> str:
    """Форматирование одной задачи."""
    status_icon = "✅" if task.get('status') == 'completed' else "🔄"
    priority_icon = {
        "high": "🔴",
        "medium": "🟡",
        "low": "🟢"
    }.get(task.get('priority', 'medium'), "⚪")
    
    lines = []
    if index is not None:
        lines.append(f"{Colors.BOLD}[{index}]{Colors.END}")
    
    lines.append(f"  {status_icon} {priority_icon} {task.get('title')}")
    
    if task.get('description'):
        desc = task['description'][:80]
        lines.append(f"    {Colors.DIM}{desc}{Colors.END}")
    
    lines.append(f"    Статус: {task.get('status')}")
    lines.append(f"    Приоритет: {task.get('priority')}")
    
    if task.get('category_name'):
        lines.append(f"    Категория: {task.get('category_name')}")
    
    if task.get('created_at'):
        created = task['created_at'][:10]
        lines.append(f"    Создано: {created}")
    
    return "\n".join(lines)


def format_task_list(tasks: List[Dict], show_index: bool = True) -> str:
    """Форматирование списка задач."""
    if not tasks:
        return f"{Colors.YELLOW}Нет задач{Colors.END}"
    
    result = []
    for i, task in enumerate(tasks, 1):
        result.append(format_task(task, i if show_index else None))
        result.append("")
    
    return "\n".join(result)


def print_table(headers: List[str], rows: List[List], padding: int = 2):
    """Вывод таблицы."""
    if not rows:
        print(f"{Colors.YELLOW}Нет данных{Colors.END}")
        return
    
    # Вычисляем ширину колонок
    col_widths = [len(h) for h in headers]
    for row in rows:
        for i, cell in enumerate(row):
            col_widths[i] = max(col_widths[i], len(str(cell)))
    
    # Разделитель
    separator = "+" + "+".join("-" * (w + padding * 2) for w in col_widths) + "+"
    
    # Заголовки
    print(separator)
    header_row = "|" + "".join(
        f" {h.center(w + padding * 2 - 2)} " for h, w in zip(headers, col_widths)
    ) + "|"
    print(header_row)
    print(separator)
    
    # Данные
    for row in rows:
        data_row = "|" + "".join(
            f" {str(cell).ljust(w + padding * 2 - 2)} " for cell, w in zip(row, col_widths)
        ) + "|"
        print(data_row)
    
    print(separator)


# ============================================================
# КОМАНДЫ
# ============================================================

def cmd_config(args):
    """Управление конфигурацией."""
    config = load_config()
    
    if args.show:
        print("Текущая конфигурация:")
        for key, value in config.items():
            print(f"  {key}: {value}")
        return
    
    if args.set:
        for item in args.set:
            if '=' not in item:
                print(f"{Colors.RED}Ошибка: используйте формат key=value{Colors.END}")
                return
            key, value = item.split('=', 1)
            config[key] = value
        save_config(config)
        print(f"{Colors.GREEN}Конфигурация сохранена{Colors.END}")


def cmd_register(args):
    """Регистрация пользователя."""
    client = APIClient()
    
    try:
        result = client.post("/auth/register", {
            "username": args.username,
            "email": args.email,
            "password": args.password,
            "confirm_password": args.password
        })
        print(f"{Colors.GREEN}✅ Пользователь {args.username} зарегистрирован{Colors.END}")
    except requests.exceptions.HTTPError as e:
        error = e.response.json()
        print(f"{Colors.RED}❌ Ошибка: {error.get('detail', str(e))}{Colors.END}")


def cmd_login(args):
    """Вход в систему."""
    client = APIClient()
    
    try:
        result = client.post("/auth/login", {
            "username": args.username,
            "password": args.password
        })
        save_token(result['access_token'])
        print(f"{Colors.GREEN}✅ Добро пожаловать, {args.username}!{Colors.END}")
        print(f"   Токен сохранён в {TOKEN_FILE}")
    except requests.exceptions.HTTPError as e:
        print(f"{Colors.RED}❌ Ошибка входа: неверное имя пользователя или пароль{Colors.END}")


def cmd_logout(args):
    """Выход из системы."""
    clear_token()
    print(f"{Colors.GREEN}✅ Вы вышли из системы{Colors.END}")


def cmd_task_create(args):
    """Создание задачи."""
    client = APIClient()
    
    data = {
        "title": args.title,
        "priority": args.priority,
        "description": args.description
    }
    
    if args.category:
        data["category_id"] = args.category
    
    try:
        result = client.post("/tasks/", data)
        print(f"{Colors.GREEN}✅ Задача создана:{Colors.END}")
        print(format_task(result))
    except requests.exceptions.HTTPError as e:
        print(f"{Colors.RED}❌ Ошибка: {e}{Colors.END}")


def cmd_task_list(args):
    """Список задач."""
    client = APIClient()
    
    params = {
        "page": args.page,
        "limit": args.limit,
    }
    
    if args.status:
        params["status"] = args.status
    if args.priority:
        params["priority"] = args.priority
    
    try:
        result = client.get("/tasks/", params)
        tasks = result.get('data', [])
        
        if args.table:
            # Табличный вывод
            headers = ["ID", "Заголовок", "Статус", "Приоритет"]
            rows = [
                [t['id'], t['title'][:30], t['status'], t['priority']]
                for t in tasks
            ]
            print_table(headers, rows)
        else:
            # Детальный вывод
            print(format_task_list(tasks))
        
        # Информация о пагинации
        print(f"\n{Colors.DIM}Страница {args.page}/{result.get('pages', 1)} | "
              f"Всего задач: {result.get('total', 0)}{Colors.END}")
        
    except requests.exceptions.HTTPError as e:
        if e.response.status_code == 401:
            print(f"{Colors.RED}❌ Не авторизован. Выполните 'taskcli login'{Colors.END}")
        else:
            print(f"{Colors.RED}❌ Ошибка: {e}{Colors.END}")


def cmd_task_get(args):
    """Получение задачи по ID."""
    client = APIClient()
    
    try:
        result = client.get(f"/tasks/{args.task_id}")
        print(format_task(result))
    except requests.exceptions.HTTPError as e:
        if e.response.status_code == 404:
            print(f"{Colors.RED}❌ Задача с ID {args.task_id} не найдена{Colors.END}")
        else:
            print(f"{Colors.RED}❌ Ошибка: {e}{Colors.END}")


def cmd_task_update(args):
    """Обновление задачи."""
    client = APIClient()
    
    data = {}
    if args.title:
        data["title"] = args.title
    if args.description:
        data["description"] = args.description
    if args.priority:
        data["priority"] = args.priority
    if args.status:
        data["status"] = args.status
    
    if not data:
        print(f"{Colors.YELLOW}Нет данных для обновления{Colors.END}")
        return
    
    try:
        result = client.put(f"/tasks/{args.task_id}", data)
        print(f"{Colors.GREEN}✅ Задача обновлена:{Colors.END}")
        print(format_task(result))
    except requests.exceptions.HTTPError as e:
        print(f"{Colors.RED}❌ Ошибка: {e}{Colors.END}")


def cmd_task_delete(args):
    """Удаление задачи."""
    client = APIClient()
    
    if not args.force:
        response = input(f"Удалить задачу {args.task_id}? [y/N]: ")
        if response.lower() != 'y':
            print("Отменено")
            return
    
    try:
        client.delete(f"/tasks/{args.task_id}")
        print(f"{Colors.GREEN}✅ Задача {args.task_id} удалена{Colors.END}")
    except requests.exceptions.HTTPError as e:
        print(f"{Colors.RED}❌ Ошибка: {e}{Colors.END}")


def cmd_task_complete(args):
    """Отметка задачи как выполненной."""
    client = APIClient()
    
    try:
        result = client.post(f"/tasks/{args.task_id}/complete", {})
        print(f"{Colors.GREEN}✅ Задача отмечена как выполненная:{Colors.END}")
        print(format_task(result))
    except requests.exceptions.HTTPError as e:
        print(f"{Colors.RED}❌ Ошибка: {e}{Colors.END}")


def cmd_task_stats(args):
    """Статистика по задачам."""
    client = APIClient()
    
    try:
        result = client.get("/tasks/stats/my")
        print(f"{Colors.BOLD}📊 Статистика задач{Colors.END}")
        print(f"  Всего задач: {result.get('total', 0)}")
        print(f"  Выполнено: {result.get('completed', 0)}")
        print(f"  В работе: {result.get('pending', 0)}")
        print(f"  Прогресс: {result.get('completion_rate', 0)}%")
    except requests.exceptions.HTTPError as e:
        print(f"{Colors.RED}❌ Ошибка: {e}{Colors.END}")


# ============================================================
# ОСНОВНАЯ ФУНКЦИЯ (argparse)
# ============================================================

def create_parser() -> argparse.ArgumentParser:
    """Создание парсера аргументов."""
    
    # Главный парсер
    parser = argparse.ArgumentParser(
        prog="taskcli",
        description="CLI для управления задачами",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog=textwrap.dedent(f"""
        {Colors.GREEN}Примеры:{Colors.END}
          taskcli login --username ivan --password secret
          taskcli task create --title "Изучить FastAPI" --priority high
          taskcli task list --status pending --limit 10
          taskcli task complete 1
          taskcli task stats
        """)
    )
    
    # Глобальные опции
    parser.add_argument(
        "--config", "-c",
        help="Путь к файлу конфигурации"
    )
    parser.add_argument(
        "--verbose", "-v",
        action="store_true",
        help="Подробный вывод"
    )
    parser.add_argument(
        "--quiet", "-q",
        action="store_true",
        help="Минимальный вывод"
    )
    
    # Подкоманды
    subparsers = parser.add_subparsers(dest="command", help="Команды")
    
    # config
    config_parser = subparsers.add_parser("config", help="Управление конфигурацией")
    config_parser.add_argument("--show", action="store_true", help="Показать конфигурацию")
    config_parser.add_argument("--set", nargs="+", metavar="KEY=VALUE", help="Установить параметры")
    
    # auth
    register_parser = subparsers.add_parser("register", help="Регистрация")
    register_parser.add_argument("--username", "-u", required=True, help="Имя пользователя")
    register_parser.add_argument("--email", "-e", required=True, help="Email")
    register_parser.add_argument("--password", "-p", required=True, help="Пароль")
    
    login_parser = subparsers.add_parser("login", help="Вход")
    login_parser.add_argument("--username", "-u", required=True, help="Имя пользователя")
    login_parser.add_argument("--password", "-p", required=True, help="Пароль")
    
    logout_parser = subparsers.add_parser("logout", help="Выход")
    
    # task
    task_parser = subparsers.add_parser("task", help="Управление задачами")
    task_subparsers = task_parser.add_subparsers(dest="task_command", help="Команды задач")
    
    # task create
    create_parser = task_subparsers.add_parser("create", help="Создать задачу")
    create_parser.add_argument("--title", "-t", required=True, help="Название задачи")
    create_parser.add_argument("--description", "-d", help="Описание")
    create_parser.add_argument("--priority", choices=["low", "medium", "high"], default="medium", help="Приоритет")
    create_parser.add_argument("--category", "-c", type=int, help="ID категории")
    
    # task list
    list_parser = task_subparsers.add_parser("list", help="Список задач")
    list_parser.add_argument("--page", type=int, default=1, help="Номер страницы")
    list_parser.add_argument("--limit", "-l", type=int, default=10, help="Записей на странице")
    list_parser.add_argument("--status", choices=["pending", "in_progress", "completed"], help="Статус")
    list_parser.add_argument("--priority", choices=["low", "medium", "high"], help="Приоритет")
    list_parser.add_argument("--table", action="store_true", help="Табличный вывод")
    
    # task get
    get_parser = task_subparsers.add_parser("get", help="Получить задачу по ID")
    get_parser.add_argument("task_id", type=int, help="ID задачи")
    
    # task update
    update_parser = task_subparsers.add_parser("update", help="Обновить задачу")
    update_parser.add_argument("task_id", type=int, help="ID задачи")
    update_parser.add_argument("--title", "-t", help="Новое название")
    update_parser.add_argument("--description", "-d", help="Новое описание")
    update_parser.add_argument("--priority", choices=["low", "medium", "high"], help="Приоритет")
    update_parser.add_argument("--status", choices=["pending", "in_progress", "completed"], help="Статус")
    
    # task delete
    delete_parser = task_subparsers.add_parser("delete", help="Удалить задачу")
    delete_parser.add_argument("task_id", type=int, help="ID задачи")
    delete_parser.add_argument("--force", "-f", action="store_true", help="Без подтверждения")
    
    # task complete
    complete_parser = task_subparsers.add_parser("complete", help="Отметить как выполненную")
    complete_parser.add_argument("task_id", type=int, help="ID задачи")
    
    # task stats
    task_subparsers.add_parser("stats", help="Статистика")
    
    return parser


def main():
    """Главная функция."""
    parser = create_parser()
    args = parser.parse_args()
    
    # Загрузка конфигурации
    config = load_config()
    if args.config:
        print(f"Используется конфиг: {args.config}")
    
    # Обработка команд
    if args.command == "config":
        cmd_config(args)
    elif args.command == "register":
        cmd_register(args)
    elif args.command == "login":
        cmd_login(args)
    elif args.command == "logout":
        cmd_logout(args)
    elif args.command == "task":
        if args.task_command == "create":
            cmd_task_create(args)
        elif args.task_command == "list":
            cmd_task_list(args)
        elif args.task_command == "get":
            cmd_task_get(args)
        elif args.task_command == "update":
            cmd_task_update(args)
        elif args.task_command == "delete":
            cmd_task_delete(args)
        elif args.task_command == "complete":
            cmd_task_complete(args)
        elif args.task_command == "stats":
            cmd_task_stats(args)
        else:
            parser.print_help()
    else:
        parser.print_help()


if __name__ == "__main__":
    main()
Примеры использования
bash
# Справка
python taskcli.py --help
python taskcli.py task --help
python taskcli.py task create --help

# Конфигурация
python taskcli.py config --show
python taskcli.py config --set api_url=http://localhost:8000 default_limit=20

# Регистрация и вход
python taskcli.py register --username ivan --email ivan@example.com --password secret123
python taskcli.py login --username ivan --password secret123

# Управление задачами
python taskcli.py task create --title "Изучить FastAPI" --priority high
python taskcli.py task list --status pending --table
python taskcli.py task list --page 1 --limit 5
python taskcli.py task get 1
python taskcli.py task update 1 --status completed
python taskcli.py task complete 1
python taskcli.py task delete 1 --force
python taskcli.py task stats

# Выход
python taskcli.py logout
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (простые аргументы, одна команда)

Варианты 9-17: средний (подкоманды, опции)

Варианты 18-25: сложный (конфигурация, цвет, автодополнение)

Варианты 1-8 (Базовый уровень)
№	Инструмент	Команды	Особенности
1	Калькулятор	add, sub, mul, div	Позиционные аргументы
2	Конвертер	length, weight, temp	Выбор единиц
3	Генератор паролей	generate	Длина, символы
4	Таймер	start, stop, status	Время выполнения
5	Заметки	add, list, delete	Хранение в файле
6	Список задач	add, list, done	Простой TODO
7	Погода	get	Город, единицы
8	Курсы валют	rate	Базовая валюта
Варианты 9-17 (Средний уровень)
№	Инструмент	Подкоманды	Дополнительно
9	Менеджер задач	task, project, tag	Фильтрация
10	Контактная книга	contact add, list, search	Экспорт
11	Бюджет	expense add, list, report	Категории
12	Парсер логов	parse, stats, search	Регулярки
13	Резервное копирование	backup, restore, list	Компрессия
14	Мониторинг	cpu, memory, disk	Графики
15	API клиент	get, post, put, delete	Форматирование
16	Генератор отчётов	report, schedule, send	Email
17	Миграции БД	init, migrate, rollback	Версионирование
Варианты 18-25 (Сложный уровень)
№	Инструмент	Особенности
18	ETL утилита	extract, transform, load
19	Управление проектами	init, build, test, deploy
20	Система плагинов	install, remove, list
21	Анализатор кода	lint, format, test
22	Деплой скрипт	config, deploy, rollback
23	Docker менеджер	build, run, stop, logs
24	Крипто-кошелёк	balance, send, receive
25	Оркестратор	run, status, stop, scale
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	CLI не работает
3 (удовлетворительно)	Базовые аргументы, одна команда
4 (хорошо)	+ подкоманды, опции, справка
5 (отлично)	+ конфигурация, цвет, автодополнение
5. Шпаргалка
python
# === БАЗОВЫЙ ПАРСЕР ===
import argparse
parser = argparse.ArgumentParser(description="Описание")
parser.add_argument("file", help="Позиционный аргумент")
parser.add_argument("--verbose", "-v", action="store_true", help="Флаг")
parser.add_argument("--count", "-c", type=int, default=1, help="Опция с числом")
args = parser.parse_args()

# === ПОДКОМАНДЫ ===
subparsers = parser.add_subparsers(dest="command")
cmd_parser = subparsers.add_parser("command", help="Описание")
cmd_parser.add_argument("--name", required=True)

# === ОБРАБОТКА ===
if args.command == "command":
    do_something(args.name)

# === ЦВЕТНОЙ ВЫВОД ===
class Colors:
    GREEN = '\033[92m'
    RED = '\033[91m'
    END = '\033[0m'
print(f"{Colors.GREEN}Успех{Colors.END}")

# === КОНФИГУРАЦИЯ ===
import json
from pathlib import Path

CONFIG_DIR = Path.home() / ".mycli"
CONFIG_FILE = CONFIG_DIR / "config.json"

def load_config():
    if CONFIG_FILE.exists():
        with open(CONFIG_FILE) as f:
            return json.load(f)
    return {}
Карточка студента
text
ПЗ 2.50. СОЗДАНИЕ CLI-ИНТЕРФЕЙСА С ARGPARSE

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== КОМАНДЫ ===

1. _____________
2. _____________
3. _____________

=== ОПЦИИ ===

□ Позиционные аргументы
□ Опциональные аргументы (--name)
□ Флаги (--verbose)
□ Подкоманды
□ Значения по умолчанию
□ Валидация

=== ДОПОЛНИТЕЛЬНО ===

□ Цветной вывод
□ Конфигурационный файл
□ Прогресс-бар
□ Автодополнение

=== ОТЧЁТ ===

Файл: cli.py
Количество команд: _____
Количество опций: _____

Дата выполнения: _____________
