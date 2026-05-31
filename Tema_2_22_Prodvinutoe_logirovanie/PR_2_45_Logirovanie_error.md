# ПЗ 2.44. Логирование ошибок в продакшен-окружении

**Тема:** Логирование, обработка ошибок, мониторинг, отказоустойчивость

**Цель работы:**  
Научиться настраивать логирование ошибок в продакшен-окружении, обрабатывать критические ситуации, отправлять уведомления об ошибках, обеспечивать отказоустойчивость системы логирования.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+

> Главная мысль: **В продакшене каждая ошибка — это потенциальная проблема для пользователя. Хорошее логирование ошибок — это первый шаг к их устранению.**

---

## Содержание

1. [Теоретическая справка](#1-теоретическая-справка)
2. [Нулевой вариант (эталонный)](#2-нулевой-вариант-эталонный)
3. [25 вариантов практической работы](#3-25-вариантов-практической-работы)
4. [Критерии оценки](#4-критерии-оценки)
5. [Шпаргалка](#5-шпаргалка)

---

## 1. Теоретическая справка

### 1.1. Особенности продакшен-логирования
┌─────────────────────────────────────────────────────────────────────────────┐
│ ПРОДАКШЕН-ЛОГИРОВАНИЕ ОШИБОК │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ ТРЕБОВАНИЯ К ЛОГАМ │ │
│ │ │ │
│ │ ✓ Безопасность — никаких паролей, токенов, персональных данных │ │
│ │ ✓ Производительность — логи не должны замедлять приложение │ │
│ │ ✓ Надёжность — логи не должны теряться при сбоях │ │
│ │ ✓ Структурированность — JSON-формат для анализа │ │
│ │ ✓ Контекст — request_id, user_id, endpoint │ │
│ │ ✓ Уровни — разные уровни для разных типов ошибок │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ КОМПОНЕНТЫ │ │
│ │ │ │
│ │ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │ │
│ │ │ Локальные │ │ Удалённые│ │Уведомле- │ │Мониторинг│ │ │
│ │ │ файлы │ │ системы │ │ния │ │ │ │ │
│ │ │ (ротация)│ │ (Sentry) │ │(Telegram)│ │ │ │ │
│ │ └──────────┘ └──────────┘ └──────────┘ └──────────┘ │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Уровни критичности для продакшена

| Уровень | Использование |
|---------|---------------|
| **CRITICAL** | Система не может работать, требуется немедленное вмешательство |
| **ERROR** | Ошибка, но система продолжает работу |
| **WARNING** | Потенциальная проблема |
| **INFO** | Важные бизнес-события (логин, заказ) |
| **DEBUG** | Не используется в продакшене (выключен) |

---

## 2. Нулевой вариант (эталонный)

**Назначение:** преподаватель выполняет этот вариант на занятии, показывая настройку логирования ошибок в продакшен-окружении.

### Техническое задание (нулевой вариант)

> Разработайте систему логирования ошибок для продакшен-окружения. Требования:
> - Многоуровневое логирование (файлы + удалённая система)
> - Отправка критических ошибок в Telegram
> - Контекстная информация (request_id, user_id, endpoint)
> - Безопасное логирование (без чувствительных данных)
> - Отказоустойчивость (логи не теряются при сбое отправки)

### Эталонная реализация

#### Файл `production_logging.py`

```python
#!/usr/bin/env python3
"""
production_logging.py — Логирование ошибок в продакшен-окружении.
"""

import os
import sys
import json
import logging
import logging.config
import traceback
import time
import threading
import queue
from pathlib import Path
from datetime import datetime
from typing import Dict, Any, Optional, Callable
from dataclasses import dataclass, field
from enum import Enum
import hashlib
import re


# ============================================================
# КОНФИГУРАЦИЯ
# ============================================================

# Маскируемые поля (не должны попадать в логи)
SENSITIVE_FIELDS = {
    'password', 'token', 'secret', 'key', 'authorization',
    'credit_card', 'cvv', 'pin', 'passphrase'
}

# Паттерны для маскировки
SENSITIVE_PATTERNS = [
    (r'(password["\s:=]+)[^"\s]+', r'\1***MASKED***'),
    (r'(token["\s:=]+)[^"\s]+', r'\1***MASKED***'),
    (r'(Authorization:?\s*Bearer\s+)[^\s]+', r'\1***MASKED***'),
]


# ============================================================
# МАСКИРОВАНИЕ ДАННЫХ
# ============================================================

class DataMasker:
    """Маскировка чувствительных данных в логах."""
    
    @staticmethod
    def mask_dict(data: Dict) -> Dict:
        """Рекурсивная маскировка чувствительных полей в словаре."""
        if not isinstance(data, dict):
            return data
        
        result = {}
        for key, value in data.items():
            key_lower = key.lower()
            
            if any(sensitive in key_lower for sensitive in SENSITIVE_FIELDS):
                result[key] = "***MASKED***"
            elif isinstance(value, dict):
                result[key] = DataMasker.mask_dict(value)
            elif isinstance(value, list):
                result[key] = [
                    DataMasker.mask_dict(item) if isinstance(item, dict) else item
                    for item in value
                ]
            else:
                result[key] = value
        
        return result
    
    @staticmethod
    def mask_string(text: str) -> str:
        """Маскировка чувствительных данных в строке."""
        if not isinstance(text, str):
            return text
        
        for pattern, replacement in SENSITIVE_PATTERNS:
            text = re.sub(pattern, replacement, text, flags=re.IGNORECASE)
        
        return text


# ============================================================
# ФОРМАТТЕРЫ
# ============================================================

class ProductionFormatter(logging.Formatter):
    """Форматтер для продакшен-логов."""
    
    def __init__(self, include_context: bool = True):
        super().__init__()
        self.include_context = include_context
    
    def format(self, record: logging.LogRecord) -> str:
        """Форматирование записи лога."""
        
        log_entry = {
            "timestamp": datetime.fromtimestamp(record.created).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": DataMasker.mask_string(record.getMessage()),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno
        }
        
        # Добавляем request_id и user_id из контекста
        log_entry["request_id"] = getattr(record, 'request_id', '-')
        log_entry["user_id"] = getattr(record, 'user_id', '-')
        log_entry["session_id"] = getattr(record, 'session_id', '-')
        
        # Добавляем endpoint для HTTP-запросов
        if hasattr(record, 'endpoint'):
            log_entry["endpoint"] = record.endpoint
        
        # Добавляем информацию об исключении
        if record.exc_info:
            log_entry["exception"] = {
                "type": record.exc_info[0].__name__,
                "message": str(record.exc_info[1]),
                "traceback": traceback.format_exception(*record.exc_info)
            }
        
        # Добавляем контекст окружения
        if self.include_context:
            log_entry["environment"] = {
                "hostname": os.uname().nodename if hasattr(os, 'uname') else 'unknown',
                "pid": os.getpid(),
                "env": os.getenv('APP_ENV', 'production')
            }
        
        return json.dumps(log_entry, ensure_ascii=False, default=str)


# ============================================================
# АСИНХРОННЫЙ ОБРАБОТЧИК (НЕ БЛОКИРУЕТ ОСНОВНОЙ ПОТОК)
# ============================================================

class AsyncQueueHandler(logging.Handler):
    """
    Асинхронный обработчик с очередью.
    Не блокирует основное приложение при записи логов.
    """
    
    def __init__(self, handlers: list, max_queue_size: int = 1000):
        super().__init__()
        self.handlers = handlers
        self.queue = queue.Queue(maxsize=max_queue_size)
        self.worker_thread = threading.Thread(target=self._process_queue, daemon=True)
        self.worker_thread.start()
    
    def emit(self, record: logging.LogRecord):
        """Добавление записи в очередь."""
        try:
            self.queue.put_nowait(record)
        except queue.Full:
            # Если очередь переполнена, пропускаем логи (не блокируем приложение)
            pass
    
    def _process_queue(self):
        """Обработка очереди в фоновом потоке."""
        while True:
            try:
                record = self.queue.get()
                for handler in self.handlers:
                    try:
                        handler.emit(record)
                    except Exception:
                        pass
            except Exception:
                pass


# ============================================================
# ОТПРАВКА УВЕДОМЛЕНИЙ В TELEGRAM
# ============================================================

class TelegramNotifier(logging.Handler):
    """
    Отправка критических ошибок в Telegram.
    """
    
    def __init__(self, bot_token: str, chat_id: str, min_level: int = logging.ERROR):
        super().__init__()
        self.bot_token = bot_token
        self.chat_id = chat_id
        self.setLevel(min_level)
        self.last_send_time = 0
        self.min_interval = 60  # не чаще 1 раза в минуту
    
    def emit(self, record: logging.LogRecord):
        """Отправка уведомления в Telegram."""
        # Ограничение частоты
        now = time.time()
        if now - self.last_send_time < self.min_interval:
            return
        
        try:
            message = self._format_message(record)
            self._send_telegram(message)
            self.last_send_time = now
        except Exception:
            pass
    
    def _format_message(self, record: logging.LogRecord) -> str:
        """Форматирование сообщения для Telegram."""
        emoji = "🔴" if record.levelname == "CRITICAL" else "🟠"
        
        message = f"""{emoji} <b>{record.levelname}</b>

<b>Модуль:</b> {record.module}
<b>Функция:</b> {record.funcName}
<b>Строка:</b> {record.lineno}
<b>Сообщение:</b> {record.getMessage()[:200]}

<b>Request ID:</b> {getattr(record, 'request_id', '-')}
<b>User ID:</b> {getattr(record, 'user_id', '-')}

<i>{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}</i>"""
        
        if record.exc_info:
            exc_type = record.exc_info[0].__name__
            message += f"\n\n<b>Исключение:</b> {exc_type}"
        
        return message
    
    def _send_telegram(self, message: str):
        """Отправка сообщения в Telegram."""
        import requests
        url = f"https://api.telegram.org/bot{self.bot_token}/sendMessage"
        data = {
            "chat_id": self.chat_id,
            "text": message,
            "parse_mode": "HTML"
        }
        try:
            requests.post(url, json=data, timeout=5)
        except Exception:
            pass


# ============================================================
# ОТПРАВКА В SENTRY (ОПЦИОНАЛЬНО)
# ============================================================

class SentryHandler(logging.Handler):
    """
    Отправка ошибок в Sentry.
    """
    
    def __init__(self, dsn: str, environment: str = "production"):
        super().__init__()
        self.dsn = dsn
        self.environment = environment
        self._init_sentry()
    
    def _init_sentry(self):
        """Инициализация Sentry."""
        try:
            import sentry_sdk
            from sentry_sdk.integrations.logging import LoggingIntegration
            
            sentry_sdk.init(
                dsn=self.dsn,
                environment=self.environment,
                integrations=[LoggingIntegration(level=logging.ERROR, event_level=logging.ERROR)]
            )
            self.sentry_available = True
        except ImportError:
            self.sentry_available = False
    
    def emit(self, record: logging.LogRecord):
        """Отправка в Sentry."""
        if not self.sentry_available:
            return
        
        try:
            import sentry_sdk
            with sentry_sdk.configure_scope() as scope:
                scope.set_tag("logger", record.name)
                scope.set_tag("request_id", getattr(record, 'request_id', '-'))
                scope.set_user({"id": getattr(record, 'user_id', '-')})
            
            if record.exc_info:
                sentry_sdk.capture_exception(record.exc_info[1])
            else:
                sentry_sdk.capture_message(record.getMessage(), level=record.levelname.lower())
        except Exception:
            pass


# ============================================================
# КОНТЕКСТНЫЙ ФИЛЬТР
# ============================================================

class ProductionContextFilter(logging.Filter):
    """
    Фильтр для добавления контекстной информации.
    """
    
    def __init__(self):
        super().__init__()
        self._local = threading.local()
    
    def set_request_id(self, request_id: str):
        self._local.request_id = request_id
    
    def set_user_id(self, user_id: str):
        self._local.user_id = user_id
    
    def set_session_id(self, session_id: str):
        self._local.session_id = session_id
    
    def set_endpoint(self, endpoint: str):
        self._local.endpoint = endpoint
    
    def filter(self, record: logging.LogRecord) -> bool:
        record.request_id = getattr(self._local, 'request_id', '-')
        record.user_id = getattr(self._local, 'user_id', '-')
        record.session_id = getattr(self._local, 'session_id', '-')
        record.endpoint = getattr(self._local, 'endpoint', '-')
        return True


# ============================================================
# ОСНОВНАЯ НАСТРОЙКА
# ============================================================

class ProductionLoggingSetup:
    """Настройка логирования для продакшен-окружения."""
    
    def __init__(self, config: Dict[str, Any] = None):
        self.config = config or self._default_config()
        self.context_filter = ProductionContextFilter()
        self._setup_logging()
    
    def _default_config(self) -> Dict:
        """Конфигурация по умолчанию."""
        return {
            "log_dir": "logs/production",
            "log_level": "INFO",
            "max_bytes": 50 * 1024 * 1024,  # 50 MB
            "backup_count": 10,
            "telegram_enabled": False,
            "telegram_bot_token": os.getenv('TELEGRAM_BOT_TOKEN', ''),
            "telegram_chat_id": os.getenv('TELEGRAM_CHAT_ID', ''),
            "sentry_enabled": False,
            "sentry_dsn": os.getenv('SENTRY_DSN', '')
        }
    
    def _setup_logging(self):
        """Настройка логгеров."""
        
        # Очищаем существующие обработчики
        root_logger = logging.getLogger()
        for handler in root_logger.handlers[:]:
            root_logger.removeHandler(handler)
        
        # Устанавливаем уровень
        root_logger.setLevel(getattr(logging, self.config['log_level']))
        
        # Создаём директорию для логов
        log_dir = Path(self.config['log_dir'])
        log_dir.mkdir(parents=True, exist_ok=True)
        
        # Форматтер
        formatter = ProductionFormatter(include_context=True)
        
        # Обработчики
        handlers = []
        
        # 1. Файловый обработчик (все логи)
        from logging.handlers import RotatingFileHandler
        file_handler = RotatingFileHandler(
            filename=log_dir / "app.log",
            maxBytes=self.config['max_bytes'],
            backupCount=self.config['backup_count'],
            encoding='utf-8'
        )
        file_handler.setLevel(logging.INFO)
        file_handler.setFormatter(formatter)
        handlers.append(file_handler)
        
        # 2. Обработчик для ошибок
        error_handler = RotatingFileHandler(
            filename=log_dir / "errors.log",
            maxBytes=self.config['max_bytes'],
            backupCount=self.config['backup_count'],
            encoding='utf-8'
        )
        error_handler.setLevel(logging.ERROR)
        error_handler.setFormatter(formatter)
        handlers.append(error_handler)
        
        # 3. Telegram уведомления (только для критических ошибок)
        if self.config['telegram_enabled'] and self.config['telegram_bot_token']:
            telegram_handler = TelegramNotifier(
                bot_token=self.config['telegram_bot_token'],
                chat_id=self.config['telegram_chat_id'],
                min_level=logging.ERROR
            )
            handlers.append(telegram_handler)
        
        # 4. Sentry
        if self.config['sentry_enabled'] and self.config['sentry_dsn']:
            sentry_handler = SentryHandler(
                dsn=self.config['sentry_dsn'],
                environment="production"
            )
            handlers.append(sentry_handler)
        
        # Асинхронная обёртка для всех обработчиков
        async_handler = AsyncQueueHandler(handlers)
        root_logger.addHandler(async_handler)
        
        # Добавляем контекстный фильтр
        root_logger.addFilter(self.context_filter)
    
    def get_logger(self, name: str) -> logging.Logger:
        """Получение логгера."""
        return logging.getLogger(name)
    
    def set_context(self, request_id: str = None, user_id: str = None, 
                    session_id: str = None, endpoint: str = None):
        """Установка контекста для текущего запроса."""
        if request_id:
            self.context_filter.set_request_id(request_id)
        if user_id:
            self.context_filter.set_user_id(str(user_id))
        if session_id:
            self.context_filter.set_session_id(session_id)
        if endpoint:
            self.context_filter.set_endpoint(endpoint)
    
    def clear_context(self):
        """Очистка контекста."""
        self.context_filter.set_request_id(None)
        self.context_filter.set_user_id(None)
        self.context_filter.set_session_id(None)
        self.context_filter.set_endpoint(None)


# ============================================================
# КЛАСС ДЛЯ ОБРАБОТКИ ОШИБОК
# ============================================================

class ProductionErrorHandler:
    """Централизованная обработка ошибок в продакшене."""
    
    def __init__(self, logger: logging.Logger):
        self.logger = logger
    
    def handle_exception(self, exc: Exception, context: Dict = None):
        """
        Обработка исключения с полным контекстом.
        
        Args:
            exc: Исключение
            context: Дополнительный контекст
        """
        error_id = hashlib.md5(str(time.time()).encode()).hexdigest()[:8]
        
        log_data = {
            "error_id": error_id,
            "error_type": type(exc).__name__,
            "error_message": str(exc),
            "context": context or {}
        }
        
        self.logger.error(
            f"Исключение {error_id}: {type(exc).__name__}: {exc}",
            exc_info=True,
            extra=log_data
        )
        
        return error_id
    
    def handle_critical(self, error_id: str, message: str):
        """Обработка критической ошибки."""
        self.logger.critical(message, extra={"error_id": error_id})


# ============================================================
# ПРИМЕР ИСПОЛЬЗОВАНИЯ
# ============================================================

def simulate_web_request(logging_setup: ProductionLoggingSetup, user_id: str, endpoint: str):
    """Симуляция обработки веб-запроса с логированием."""
    
    # Устанавливаем контекст запроса
    request_id = hashlib.md5(f"{time.time()}{user_id}".encode()).hexdigest()[:16]
    logging_setup.set_context(
        request_id=request_id,
        user_id=user_id,
        endpoint=endpoint
    )
    
    logger = logging_setup.get_logger("app.web")
    error_handler = ProductionErrorHandler(logger)
    
    logger.info(f"Обработка запроса {endpoint}", extra={"user_id": user_id})
    
    try:
        # Симуляция бизнес-логики
        if user_id == "bad_user":
            raise ValueError("Неверный формат данных")
        
        if endpoint == "/error":
            raise RuntimeError("Внутренняя ошибка сервера")
        
        logger.info("Запрос успешно обработан")
        
    except Exception as e:
        error_id = error_handler.handle_exception(e, {"endpoint": endpoint, "user_id": user_id})
        logger.error(f"Ошибка при обработке запроса", extra={"error_id": error_id})
    
    finally:
        # Очищаем контекст после запроса
        logging_setup.clear_context()


def main():
    """Демонстрация логирования ошибок в продакшене."""
    
    print("=" * 70)
    print("ЛОГИРОВАНИЕ ОШИБОК В ПРОДАКШЕН-ОКРУЖЕНИИ")
    print("=" * 70)
    
    # Настройка логирования
    config = {
        "log_dir": "logs/production_demo",
        "log_level": "DEBUG",
        "max_bytes": 1024 * 1024,  # 1 MB
        "backup_count": 3,
        "telegram_enabled": False,  # Для демонстрации отключено
        "sentry_enabled": False
    }
    
    logging_setup = ProductionLoggingSetup(config)
    logger = logging_setup.get_logger("app")
    error_handler = ProductionErrorHandler(logger)
    
    print("\n📋 1. Обычное логирование")
    print("-" * 50)
    logger.info("Приложение запущено")
    logger.warning("Предупреждение: низкий уровень памяти")
    
    print("\n🌐 2. Логирование веб-запросов с контекстом")
    print("-" * 50)
    simulate_web_request(logging_setup, "user123", "/api/users")
    simulate_web_request(logging_setup, "bad_user", "/api/profile")
    simulate_web_request(logging_setup, "admin", "/error")
    
    print("\n❌ 3. Логирование исключений")
    print("-" * 50)
    try:
        result = 10 / 0
    except ZeroDivisionError as e:
        error_id = error_handler.handle_exception(e, {"operation": "division", "value": 10})
        print(f"   Ошибка залогирована с ID: {error_id}")
    
    print("\n🔴 4. Критическая ошибка")
    print("-" * 50)
    error_handler.handle_critical("CRIT-001", "База данных недоступна")
    
    print("\n📊 5. Проверка лог-файлов")
    print("-" * 50)
    log_dir = Path("logs/production_demo")
    if log_dir.exists():
        for log_file in log_dir.glob("*.log"):
            size = log_file.stat().st_size / 1024
            print(f"   {log_file.name}: {size:.1f} KB")
    
    print("\n" + "=" * 70)
    print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
    print("=" * 70)
    print("\n📁 Логи сохранены в директории: logs/production_demo/")
    print("   - app.log — все логи")
    print("   - errors.log — только ошибки")
    print("\n💡 Для просмотра логов в реальном времени:")
    print("   tail -f logs/production_demo/app.log | jq '.'")


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
======================================================================
ЛОГИРОВАНИЕ ОШИБОК В ПРОДАКШЕН-ОКРУЖЕНИИ
======================================================================

📋 1. Обычное логирование
--------------------------------------------------
INFO: Приложение запущено
WARNING: Предупреждение: низкий уровень памяти

🌐 2. Логирование веб-запросов с контекстом
--------------------------------------------------
INFO: Обработка запроса /api/users
INFO: Запрос успешно обработан
INFO: Обработка запроса /api/profile
ERROR: Исключение 1a2b3c4d: ValueError: Неверный формат данных
INFO: Обработка запроса /error
ERROR: Исключение 5e6f7g8h: RuntimeError: Внутренняя ошибка сервера
ERROR: Ошибка при обработке запроса

❌ 3. Логирование исключений
--------------------------------------------------
   Ошибка залогирована с ID: 9i0j1k2l

🔴 4. Критическая ошибка
--------------------------------------------------
CRITICAL: База данных недоступна

📊 5. Проверка лог-файлов
--------------------------------------------------
   app.log: 4.2 KB
   errors.log: 2.1 KB

======================================================================
ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА
======================================================================

📁 Логи сохранены в директории: logs/production_demo/
   - app.log — все логи
   - errors.log — только ошибки

💡 Для просмотра логов в реальном времени:
   tail -f logs/production_demo/app.log | jq '.'
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (файловое логирование, ротация)

Варианты 9-17: средний (+ контекст, уведомления)

Варианты 18-25: сложный (+ удалённые системы, отказоустойчивость)

Варианты 1-8 (Базовый уровень)
№	Особенности	Уведомления	Ротация
1	Файловое логирование	Нет	По размеру
2	Раздельные файлы (ошибки/все)	Нет	По размеру
3	Маскировка данных	Нет	По времени
4	Контекст (user_id)	Нет	По размеру
5	JSON-формат	Нет	По размеру
6	Request ID	Нет	По времени
7	Email уведомления	Email	По размеру
8	Консоль + файл	Нет	По размеру
Варианты 9-17 (Средний уровень)
№	Особенности	Уведомления	Дополнительно
9	Асинхронное логирование	Telegram	Ротация
10	Контекст + маскировка	Email	Sentry
11	Request ID + User ID	Telegram	JSON формат
12	Отказоустойчивая очередь	Slack	Ротация
13	Группировка ошибок	Telegram	Анализ
14	Метрики ошибок	Email	Статистика
15	Алерты по частоте	Telegram	Дашборд
16	Логирование запросов	Webhook	Ротация
17	Многопоточное	Telegram	Отказоустойчивость
Варианты 18-25 (Сложный уровень)
№	Тема	Компоненты
18	Полная продакшен-система	Файлы + Telegram + Sentry
19	Микросервисное логирование	Централизованный сбор
20	Аналитика ошибок	Графики, дашборды
21	Автоматическое восстановление	Retry, fallback
22	Логирование в Kubernetes	stdout + ELK
23	HIPAA-совместимое логирование	Аудит, маскировка
24	Асинхронная обработка	RabbitMQ + Elastic
25	Корреляция ошибок	Tracing, distributed logs
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Ошибки не логируются
3 (удовлетворительно)	Базовое логирование в файл
4 (хорошо)	+ контекст, уведомления, маскировка
5 (отлично)	+ асинхронность, отказоустойчивость, Sentry
5. Шпаргалка
python
# === ПРОДАКШЕН-НАСТРОЙКА ===
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        RotatingFileHandler('app.log', maxBytes=10_000_000, backupCount=5),
        logging.StreamHandler()
    ]
)

# === АСИНХРОННОЕ ЛОГИРОВАНИЕ ===
queue_handler = QueueHandler(queue)
listener = QueueListener(queue, *handlers)

# === МАСКИРОВКА ===
def mask_sensitive(data):
    if 'password' in data:
        data['password'] = '***MASKED***'
    return data

# === КОНТЕКСТ ===
class ContextFilter(logging.Filter):
    def filter(self, record):
        record.user_id = get_current_user_id()
        return True
Карточка студента
text
ПЗ 2.44. ЛОГИРОВАНИЕ ОШИБОК В ПРОДАКШЕН-ОКРУЖЕНИИ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== КОМПОНЕНТЫ ===

□ Файловое логирование с ротацией
□ Раздельные файлы (app.log, errors.log)
□ JSON-формат
□ Маскировка чувствительных данных
□ Контекст (request_id, user_id)
□ Асинхронная запись
□ Уведомления (Telegram/Email)
□ Sentry интеграция

=== ОТЧЁТ ===

Количество обработанных ошибок: _____
Размер лог-файлов: _____ KB
Время записи (среднее): _____ мс

Дата выполнения: _____________
