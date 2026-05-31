# ПЗ 2.44. Настройка сбора логов с разных модулей в единый файл (JSON-формат)

**Тема:** Продвинутое логирование, JSON-формат, централизованный сбор логов, структурированное логирование

**Цель работы:**  
Научиться настраивать централизованный сбор логов с разных модулей в единый JSON-файл, использовать структурированное логирование для упрощения анализа и интеграции с системами мониторинга.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+

> Главная мысль: **JSON-логи — это мост между вашим приложением и системами аналитики. Структурированные логи легко парсить, фильтровать и анализировать.**

---

## Содержание

1. [Теоретическая справка](#1-теоретическая-справка)
2. [Нулевой вариант (эталонный)](#2-нулевой-вариант-эталонный)
3. [25 вариантов практической работы](#3-25-вариантов-практической-работы)
4. [Критерии оценки](#4-критерии-оценки)
5. [Шпаргалка](#5-шпаргалка)

---

## 1. Теоретическая справка

### 1.1. Преимущества JSON-формата для логов

| Характеристика | Текстовые логи | JSON-логи |
|----------------|----------------|-----------|
| **Читаемость** | Хорошая для человека | Хорошая для машин |
| **Парсинг** | Сложный (regex) | Простой (json.loads) |
| **Фильтрация** | Трудоёмкая | Лёгкая (jq) |
| **Интеграция** | Сложная | Простая (ELK, Splunk) |
| **Размер** | Меньше | Больше (из-за скобок) |

### 1.2. Структура JSON-лога

```json
{
    "timestamp": "2024-01-15T14:30:25.123456",
    "level": "INFO",
    "logger": "app.database",
    "module": "database",
    "function": "connect",
    "line": 42,
    "message": "Подключение к БД установлено",
    "user_id": "user123",
    "session_id": "abc-123",
    "execution_time": 0.15,
    "context": {
        "db_host": "localhost",
        "db_name": "myapp"
    }
}
1.3. Компоненты системы сбора логов
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    СБОР ЛОГОВ В JSON-ФОРМАТЕ                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         ПРИЛОЖЕНИЕ                                  │   │
│   │                                                                     │   │
│   │  ┌──────────┐    ┌──────────┐    ┌──────────┐                      │   │
│   │  │ Модуль A │    │ Модуль B │    │ Модуль C │                      │   │
│   │  │(logger A)│    │(logger B)│    │(logger C)│                      │   │
│   │  └────┬─────┘    └────┬─────┘    └────┬─────┘                      │   │
│   │       │               │               │                             │   │
│   │       └───────────────┼───────────────┘                             │   │
│   │                       │                                             │   │
│   │                       ▼                                             │   │
│   │              ┌─────────────────┐                                    │   │
│   │              │ JSON Formatter  │                                    │   │
│   │              └────────┬────────┘                                    │   │
│   │                       │                                             │   │
│   │                       ▼                                             │   │
│   │              ┌─────────────────┐                                    │   │
│   │              │   FileHandler   │                                    │   │
│   │              │   (app.jsonl)   │                                    │   │
│   │              └────────┬────────┘                                    │   │
│   └───────────────────────┼─────────────────────────────────────────────┘   │
│                           │                                                 │
│                           ▼                                                 │
│               ┌─────────────────────────┐                                  │
│               │     app.jsonl           │                                  │
│               │   (JSON Lines format)   │                                  │
│               └─────────────────────────┘                                  │
│                           │                                                 │
│                           ▼                                                 │
│               ┌─────────────────────────┐                                  │
│               │   Аналитика / ELK       │                                  │
│               │   (Elasticsearch, etc)  │                                  │
│               └─────────────────────────┘                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая настройку сбора логов в JSON-формате.

Техническое задание (нулевой вариант)
Разработайте систему сбора логов с разных модулей приложения в единый JSON-файл. Требования:

JSON Lines формат (каждая строка — отдельный JSON-объект)

Включение контекстной информации (user_id, session_id)

Разные уровни логирования для разных модулей

Ротация JSON-логов по размеру

Возможность фильтрации при чтении

Эталонная реализация
Файл json_logging_config.py
python
#!/usr/bin/env python3
"""
json_logging_config.py — Настройка сбора логов в JSON-формате.
"""

import json
import logging
import logging.config
from pathlib import Path
from datetime import datetime
import sys
import os
from typing import Dict, Any, Optional
import traceback


# ============================================================
# JSON ФОРМАТТЕР
# ============================================================

class JSONFormatter(logging.Formatter):
    """
    Форматтер для вывода логов в JSON-формате (JSON Lines).
    Каждая строка — отдельный JSON-объект.
    """
    
    def __init__(self, include_context: bool = True, **kwargs):
        super().__init__(**kwargs)
        self.include_context = include_context
    
    def format(self, record: logging.LogRecord) -> str:
        """Форматирование записи лога в JSON."""
        
        # Базовые поля
        log_entry = {
            "timestamp": datetime.fromtimestamp(record.created).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno,
            "message": record.getMessage(),
            "process": record.process,
            "process_name": record.processName,
            "thread": record.thread,
            "thread_name": record.threadName
        }
        
        # Добавляем информацию об исключении, если есть
        if record.exc_info:
            log_entry["exception"] = {
                "type": record.exc_info[0].__name__,
                "message": str(record.exc_info[1]),
                "traceback": traceback.format_exception(*record.exc_info)
            }
        
        # Добавляем пользовательские атрибуты
        for key, value in record.__dict__.items():
            if key not in log_entry and not key.startswith('_'):
                # Пропускаем несериализуемые объекты
                try:
                    json.dumps(value)
                    log_entry[key] = value
                except (TypeError, ValueError):
                    log_entry[key] = str(value)
        
        # Добавляем контекст, если нужно
        if self.include_context:
            log_entry["context"] = {
                "hostname": os.uname().nodename if hasattr(os, 'uname') else 'unknown',
                "pid": os.getpid(),
                "cwd": os.getcwd()
            }
        
        # Возвращаем JSON строку
        return json.dumps(log_entry, ensure_ascii=False, default=str)


# ============================================================
# JSON LINES ОБРАБОТЧИК (С РОТАЦИЕЙ)
# ============================================================

class JSONLinesRotatingHandler(logging.Handler):
    """
    Обработчик для записи логов в JSON Lines формат с ротацией.
    """
    
    def __init__(self, filename: str, max_bytes: int = 10_000_000, backup_count: int = 5, encoding: str = 'utf-8'):
        super().__init__()
        self.filename = Path(filename)
        self.max_bytes = max_bytes
        self.backup_count = backup_count
        self.encoding = encoding
        self.formatter = JSONFormatter()
        
        # Создаём директорию
        self.filename.parent.mkdir(parents=True, exist_ok=True)
        
        # Открываем файл
        self._open_file()
    
    def _open_file(self):
        """Открытие файла для записи."""
        self.stream = open(self.filename, 'a', encoding=self.encoding)
    
    def _should_rollover(self) -> bool:
        """Проверка необходимости ротации."""
        if not self.filename.exists():
            return False
        return self.filename.stat().st_size >= self.max_bytes
    
    def _do_rollover(self):
        """Выполнение ротации."""
        self.stream.close()
        
        # Переименовываем файлы
        for i in range(self.backup_count - 1, 0, -1):
            src = self.filename.with_suffix(f'.{i}.jsonl')
            dst = self.filename.with_suffix(f'.{i + 1}.jsonl')
            if src.exists():
                src.rename(dst)
        
        # Переименовываем текущий файл
        if self.filename.exists():
            self.filename.rename(self.filename.with_suffix('.1.jsonl'))
        
        # Открываем новый файл
        self._open_file()
    
    def emit(self, record: logging.LogRecord):
        """Запись записи в файл."""
        try:
            if self._should_rollover():
                self._do_rollover()
            
            msg = self.formatter.format(record)
            self.stream.write(msg + '\n')
            self.stream.flush()
        except Exception:
            self.handleError(record)
    
    def close(self):
        """Закрытие обработчика."""
        if self.stream:
            self.stream.close()
        super().close()


# ============================================================
# КОНТЕКСТНЫЙ ФИЛЬТР
# ============================================================

class ContextFilter(logging.Filter):
    """
    Фильтр для добавления контекстной информации.
    """
    
    def __init__(self):
        super().__init__()
        self._local = threading.local()
    
    def set_user_id(self, user_id: str):
        self._local.user_id = user_id
    
    def set_session_id(self, session_id: str):
        self._local.session_id = session_id
    
    def set_request_id(self, request_id: str):
        self._local.request_id = request_id
    
    def filter(self, record):
        # Добавляем пользовательские атрибуты
        record.user_id = getattr(self._local, 'user_id', 'anonymous')
        record.session_id = getattr(self._local, 'session_id', '-')
        record.request_id = getattr(self._local, 'request_id', '-')
        return True


# ============================================================
# НАСТРОЙКА ЛОГГЕРОВ
# ============================================================

import threading


class JSONLoggingConfig:
    """Настройка централизованного сбора логов в JSON."""
    
    def __init__(self, log_dir: str = "logs/json_logs", max_mb: int = 10):
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(parents=True, exist_ok=True)
        self.max_bytes = max_mb * 1024 * 1024
        self.context_filter = ContextFilter()
        self._setup_logging()
    
    def _setup_logging(self):
        """Настройка всех логгеров."""
        
        # Очищаем существующие обработчики
        root_logger = logging.getLogger()
        for handler in root_logger.handlers[:]:
            root_logger.removeHandler(handler)
        
        # Устанавливаем минимальный уровень
        root_logger.setLevel(logging.DEBUG)
        
        # Основной JSON-обработчик (все логи)
        main_handler = JSONLinesRotatingHandler(
            filename=self.log_dir / "all_logs.jsonl",
            max_bytes=self.max_bytes,
            backup_count=10,
            encoding='utf-8'
        )
        main_handler.setLevel(logging.DEBUG)
        root_logger.addHandler(main_handler)
        
        # Отдельный обработчик для ошибок
        error_handler = JSONLinesRotatingHandler(
            filename=self.log_dir / "errors.jsonl",
            max_bytes=self.max_bytes,
            backup_count=5,
            encoding='utf-8'
        )
        error_handler.setLevel(logging.ERROR)
        root_logger.addHandler(error_handler)
        
        # Консольный обработчик (для разработки)
        console_handler = logging.StreamHandler(sys.stdout)
        console_handler.setLevel(logging.INFO)
        console_handler.setFormatter(logging.Formatter('%(levelname)s: %(message)s'))
        root_logger.addHandler(console_handler)
        
        # Добавляем контекстный фильтр
        root_logger.addFilter(self.context_filter)
        
        # Настройка логгеров разных модулей
        self._setup_module_loggers()
    
    def _setup_module_loggers(self):
        """Настройка логгеров для разных модулей."""
        
        # Модуль авторизации (INFO)
        auth_logger = logging.getLogger('app.auth')
        auth_logger.setLevel(logging.INFO)
        
        # Модуль базы данных (DEBUG)
        db_logger = logging.getLogger('app.database')
        db_logger.setLevel(logging.DEBUG)
        
        # Модуль API (WARNING)
        api_logger = logging.getLogger('app.api')
        api_logger.setLevel(logging.WARNING)
        
        # Модуль утилит (INFO)
        utils_logger = logging.getLogger('app.utils')
        utils_logger.setLevel(logging.INFO)
    
    def get_logger(self, name: str) -> logging.Logger:
        """Получение логгера."""
        return logging.getLogger(name)
    
    def set_user_context(self, user_id: str, session_id: str = None, request_id: str = None):
        """Установка контекста пользователя."""
        self.context_filter.set_user_id(user_id)
        if session_id:
            self.context_filter.set_session_id(session_id)
        if request_id:
            self.context_filter.set_request_id(request_id)
    
    def clear_user_context(self):
        """Очистка контекста пользователя."""
        self.context_filter.set_user_id('anonymous')
        self.context_filter.set_session_id('-')
        self.context_filter.set_request_id('-')
    
    def read_logs(self, log_file: str = "all_logs.jsonl", 
                  level: Optional[str] = None,
                  logger_name: Optional[str] = None,
                  limit: int = 100) -> list:
        """
        Чтение и фильтрация JSON-логов.
        
        Args:
            log_file: Имя файла
            level: Фильтр по уровню (INFO, ERROR и т.д.)
            logger_name: Фильтр по имени логгера
            limit: Максимальное количество записей
        
        Returns:
            Список отфильтрованных записей
        """
        log_path = self.log_dir / log_file
        if not log_path.exists():
            return []
        
        results = []
        with open(log_path, 'r', encoding='utf-8') as f:
            for line in f:
                if len(results) >= limit:
                    break
                
                try:
                    log_entry = json.loads(line.strip())
                    
                    # Применяем фильтры
                    if level and log_entry.get('level') != level:
                        continue
                    if logger_name and not log_entry.get('logger', '').startswith(logger_name):
                        continue
                    
                    results.append(log_entry)
                except json.JSONDecodeError:
                    continue
        
        return results


# ============================================================
# МОДУЛИ ПРИЛОЖЕНИЯ
# ============================================================

class AuthModule:
    """Модуль авторизации."""
    
    def __init__(self, logger: logging.Logger):
        self.logger = logger
    
    def login(self, user_id: str, password: str) -> bool:
        self.logger.info(f"Попытка входа", extra={"user_id": user_id})
        
        if len(password) < 6:
            self.logger.warning(f"Слабый пароль", extra={"user_id": user_id})
            return False
        
        self.logger.info(f"Успешный вход", extra={"user_id": user_id})
        return True


class DatabaseModule:
    """Модуль базы данных."""
    
    def __init__(self, logger: logging.Logger):
        self.logger = logger
    
    def query(self, sql: str):
        self.logger.debug(f"Выполнение запроса", extra={"sql": sql[:100]})
        # Имитация выполнения
        self.logger.debug("Запрос выполнен успешно")
    
    def connect(self):
        self.logger.info("Подключение к БД")


class APIModule:
    """API-клиент."""
    
    def __init__(self, logger: logging.Logger):
        self.logger = logger
    
    def call(self, endpoint: str):
        self.logger.debug(f"Вызов API", extra={"endpoint": endpoint})
        
        import random
        if random.random() < 0.2:
            self.logger.error(f"Ошибка API", extra={"endpoint": endpoint})
            return False
        
        self.logger.info("API вызов успешен")
        return True


# ============================================================
# ГЛАВНАЯ ФУНКЦИЯ
# ============================================================

def main():
    """Демонстрация работы JSON-логирования."""
    
    print("=" * 70)
    print("СБОР ЛОГОВ В JSON-ФОРМАТЕ С РАЗНЫХ МОДУЛЕЙ")
    print("=" * 70)
    
    # Настройка логирования
    log_config = JSONLoggingConfig(log_dir="logs/json_demo", max_mb=1)
    
    # Получение логгеров
    root_logger = log_config.get_logger('app')
    auth_logger = log_config.get_logger('app.auth')
    db_logger = log_config.get_logger('app.database')
    api_logger = log_config.get_logger('app.api')
    
    # Создание модулей
    auth = AuthModule(auth_logger)
    db = DatabaseModule(db_logger)
    api = APIModule(api_logger)
    
    print("\n📋 1. Логирование без контекста")
    print("-" * 50)
    root_logger.info("Приложение запущено")
    root_logger.warning("Предупреждение")
    
    print("\n🔐 2. Логирование с контекстом пользователя")
    print("-" * 50)
    log_config.set_user_context(user_id="user123", session_id="sess_abc")
    
    auth.login("user123", "weak")
    auth.login("user456", "strong_password")
    
    print("\n💾 3. Логирование БД")
    print("-" * 50)
    db.connect()
    db.query("SELECT * FROM users WHERE id = 1")
    
    print("\n🌐 4. Логирование API")
    print("-" * 50)
    for i in range(10):
        api.call(f"/api/users/{i}")
    
    # Очистка контекста
    log_config.clear_user_context()
    print("\n👤 5. Логирование без контекста")
    print("-" * 50)
    root_logger.info("Операция без пользователя")
    
    # ============================================================
    # ЧТЕНИЕ И АНАЛИЗ ЛОГОВ
    # ============================================================
    
    print("\n" + "=" * 70)
    print("АНАЛИЗ СОБРАННЫХ JSON-ЛОГОВ")
    print("=" * 70)
    
    # Чтение всех логов
    print("\n📊 Последние 5 записей:")
    all_logs = log_config.read_logs(limit=5)
    for log_entry in all_logs:
        print(f"   [{log_entry['level']}] {log_entry['logger']}: {log_entry['message'][:50]}...")
    
    # Чтение ошибок
    print("\n❌ Только ошибки:")
    errors = log_config.read_logs(level="ERROR", limit=5)
    for error in errors:
        print(f"   {error['logger']}: {error['message']}")
    
    # Чтение логов по модулю
    print("\n📁 Только логи БД:")
    db_logs = log_config.read_logs(logger_name="app.database", limit=5)
    for log_entry in db_logs:
        print(f"   [{log_entry['level']}] {log_entry['message']}")
    
    # Статистика по уровням
    print("\n📈 Статистика по уровням:")
    all_logs_full = log_config.read_logs(limit=1000)
    stats = {}
    for log_entry in all_logs_full:
        level = log_entry['level']
        stats[level] = stats.get(level, 0) + 1
    
    for level, count in sorted(stats.items()):
        print(f"   {level}: {count}")
    
    print("\n" + "=" * 70)
    print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
    print("=" * 70)
    print(f"\n📁 JSON-логи сохранены в директории: logs/json_demo/")
    print("   - all_logs.jsonl — все логи")
    print("   - errors.jsonl — только ошибки")
    print("\n💡 Пример анализа с помощью jq:")
    print("   cat logs/json_demo/all_logs.jsonl | jq '.level, .message'")
    print("   cat logs/json_demo/all_logs.jsonl | jq 'select(.level == \"ERROR\")'")


if __name__ == "__main__":
    main()
Пример содержимого JSON-лога
json
{"timestamp": "2024-01-15T14:30:25.123456", "level": "INFO", "logger": "app", "module": "json_logging_config", "function": "main", "line": 420, "message": "Приложение запущено", "process": 12345, "process_name": "MainProcess", "thread": 123456789, "thread_name": "MainThread", "user_id": "anonymous", "session_id": "-", "request_id": "-", "context": {"hostname": "myhost", "pid": 12345, "cwd": "/home/user/project"}}
{"timestamp": "2024-01-15T14:30:25.123567", "level": "INFO", "logger": "app.auth", "module": "json_logging_config", "function": "login", "line": 380, "message": "Попытка входа", "process": 12345, "process_name": "MainProcess", "thread": 123456789, "thread_name": "MainThread", "user_id": "user123", "session_id": "sess_abc", "request_id": "-", "context": {"hostname": "myhost", "pid": 12345, "cwd": "/home/user/project"}}
Ожидаемый вывод
text
======================================================================
СБОР ЛОГОВ В JSON-ФОРМАТЕ С РАЗНЫХ МОДУЛЕЙ
======================================================================

📋 1. Логирование без контекста
--------------------------------------------------
INFO: Приложение запущено
WARNING: Предупреждение

🔐 2. Логирование с контекстом пользователя
--------------------------------------------------
INFO: Попытка входа
WARNING: Слабый пароль
INFO: Попытка входа
INFO: Успешный вход

💾 3. Логирование БД
--------------------------------------------------
INFO: Подключение к БД
DEBUG: Выполнение запроса
DEBUG: Запрос выполнен успешно

🌐 4. Логирование API
--------------------------------------------------
DEBUG: Вызов API
INFO: API вызов успешен
...

👤 5. Логирование без контекста
--------------------------------------------------
INFO: Операция без пользователя

======================================================================
АНАЛИЗ СОБРАННЫХ JSON-ЛОГОВ
======================================================================

📊 Последние 5 записей:
   [INFO] app: Приложение запущено...
   [WARNING] app: Предупреждение...
   [INFO] app.auth: Попытка входа...
   [WARNING] app.auth: Слабый пароль...
   [INFO] app.auth: Попытка входа...

❌ Только ошибки:
   app.api: Ошибка API
   app.api: Ошибка API

📁 Только логи БД:
   [INFO] Подключение к БД
   [DEBUG] Выполнение запроса
   [DEBUG] Запрос выполнен успешно

📈 Статистика по уровням:
   DEBUG: 15
   ERROR: 2
   INFO: 12
   WARNING: 2

======================================================================
ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА
======================================================================

📁 JSON-логи сохранены в директории: logs/json_demo/
   - all_logs.jsonl — все логи
   - errors.jsonl — только ошибки

💡 Пример анализа с помощью jq:
   cat logs/json_demo/all_logs.jsonl | jq '.level, .message'
   cat logs/json_demo/all_logs.jsonl | jq 'select(.level == "ERROR")'
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (один JSON-файл, простые модули)

Варианты 9-17: средний (контекст, фильтрация, ротация)

Варианты 18-25: сложный (несколько файлов, агрегация, анализ)

Варианты 1-8 (Базовый уровень)
№	Модули	JSON-формат	Ротация
1	app, db, utils	Базовый	Нет
2	auth, data, log	Только основные поля	Нет
3	web, api, cache	С timestamp	Нет
4	main, worker, queue	Уровень+модуль	По размеру
5	producer, consumer	Полный	По размеру
6	parser, saver	С исключениями	Нет
7	fetcher, validator	С контекстом	По размеру
8	scheduler, executor	Базовый	Нет
Варианты 9-17 (Средний уровень)
№	Модули	Дополнительные требования
9	app, auth, db, api	Контекст (user_id, session_id)
10	web, auth, db, cache	Ротация по размеру и времени
11	server, handler, service	Отдельный файл для ошибок
12	controller, repository, model	JSON Lines формат
13	extractor, transformer, loader	Чтение и фильтрация логов
14	monitor, alert, metric	Статистика по уровням
15	cli, config, plugin	Контекст выполнения
16	collector, aggregator	Агрегация по модулям
17	producer, consumer, broker	Анализ производительности
Варианты 18-25 (Сложный уровень)
№	Тема	Особенности
18	Микросервисная архитектура	Несколько JSON-файлов, агрегация
19	Веб-приложение	Request ID, трассировка
20	ETL-пайплайн	Логирование этапов, метрики
21	Система мониторинга	Сбор метрик, алерты
22	API-шлюз	Логирование запросов/ответов
23	Распределённая система	Correlation ID, tracing
24	Big Data обработка	Потоковая запись, сжатие
25	Cloud-native приложение	Интеграция с ELK/Splunk
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	JSON-логи не собираются
3 (удовлетворительно)	Логи собираются в JSON, базовый формат
4 (хорошо)	+ контекст, ротация, фильтрация
5 (отлично)	+ анализ логов, статистика, JSON Lines
5. Шпаргалка
python
# === JSON ФОРМАТТЕР ===
class JSONFormatter(logging.Formatter):
    def format(self, record):
        return json.dumps({
            "timestamp": datetime.now().isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage()
        })

# === JSON LINES ОБРАБОТЧИК ===
class JSONLinesHandler(logging.Handler):
    def emit(self, record):
        with open('logs.jsonl', 'a') as f:
            f.write(self.format(record) + '\n')

# === ЧТЕНИЕ ЛОГОВ ===
import json
def read_logs(filepath):
    with open(filepath, 'r') as f:
        return [json.loads(line) for line in f]

# === АНАЛИЗ С jq ===
# cat logs.jsonl | jq '.level'
# cat logs.jsonl | jq 'select(.level == "ERROR")'
Карточка студента
text
ПЗ 2.43. НАСТРОЙКА СБОРА ЛОГОВ В JSON-ФОРМАТЕ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== МОДУЛИ ===

1. _____________
2. _____________
3. _____________
4. _____________
5. _____________

=== JSON-ФОРМАТ ===

□ Базовые поля (timestamp, level, logger, message)
□ Исключения (traceback)
□ Контекст (user_id, session_id)
□ Performance (execution_time)
□ Процесс/поток

=== КОМПОНЕНТЫ ===

□ JSONFormatter
□ JSONLinesHandler
□ Ротация по размеру
□ Отдельный файл для ошибок
□ Чтение и фильтрация логов
□ Статистика

=== ОТЧЁТ ===

Количество записей: _____
Размер JSON-файла: _____ KB
Типичная запись: _____________

Дата выполнения: _____________
