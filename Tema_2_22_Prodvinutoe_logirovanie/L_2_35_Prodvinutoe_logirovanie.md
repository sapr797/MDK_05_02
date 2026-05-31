# Тема 2.35. Продвинутое логирование: иерархия логгеров, фильтры

**Цель лекции:**  
Изучить продвинутые возможности модуля logging: создание иерархии логгеров, использование фильтров для гибкой маршрутизации сообщений, настройка нескольких обработчиков с разными уровнями и форматами.

> Главная мысль: **Правильно настроенное логирование — это как приборная панель самолёта: в нужный момент вы видите именно ту информацию, которая нужна.**

---

## Содержание

1. [Иерархия логгеров](#1-иерархия-логгеров)
2. [Фильтры в логировании](#2-фильтры-в-логировании)
3. [Продвинутая настройка обработчиков](#3-продвинутая-настройка-обработчиков)
4. [Кастомные форматеры](#4-кастомные-форматеры)
5. [Логирование в разных окружениях](#5-логирование-в-разных-окружениях)
6. [Практические примеры](#6-практические-примеры)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Иерархия логгеров

### 1.1. Принцип иерархии

Логгеры в Python организованы в иерархическую структуру, где имена разделяются точками.
┌─────────────────────────────────────────────────────────────────────────────┐
│ ИЕРАРХИЯ ЛОГГЕРОВ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ┌─────────────────┐ │
│ │ root │ │
│ │ (корневой) │ │
│ └────────┬────────┘ │
│ │ │
│ ┌───────────────────┼───────────────────┐ │
│ │ │ │ │
│ ▼ ▼ ▼ │
│ ┌───────────┐ ┌───────────┐ ┌───────────┐ │
│ │ app │ │ database │ │ api │ │
│ └─────┬─────┘ └───────────┘ └─────┬─────┘ │
│ │ │ │
│ ┌──────┼──────┐ │ │
│ ▼ ▼ ▼ ▼ │
│ ┌──────┐┌──────┐┌──────┐ ┌───────────┐ │
│ │models││views ││utils │ │ client │ │
│ └──────┘└──────┘└──────┘ └───────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Создание иерархии логгеров

```python
import logging

# Корневой логгер (создаётся автоматически)
root_logger = logging.getLogger()

# Логгеры приложения
app_logger = logging.getLogger('app')
db_logger = logging.getLogger('app.database')
api_logger = logging.getLogger('app.api')
models_logger = logging.getLogger('app.models')

# Логгеры наследуют настройки от родительских
print(f"Уровень app_logger: {app_logger.level}")  # 0 (NOTSET) — наследует от root
print(f"Propagate: {app_logger.propagate}")  # True — сообщения идут к родителю
1.3. Настройка иерархии
python
import logging

# Настройка корневого логгера
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Создание дочерних логгеров
app_logger = logging.getLogger('app')
db_logger = logging.getLogger('app.database')
api_logger = logging.getLogger('app.api')

# Установка разных уровней для разных логгеров
db_logger.setLevel(logging.DEBUG)    # БД логируем подробно
api_logger.setLevel(logging.WARNING)  # API — только предупреждения

# Использование
app_logger.info("Сообщение от app")        # выводится (INFO)
db_logger.debug("Отладка БД")             # выводится (DEBUG)
db_logger.info("Инфо БД")                 # выводится (INFO)
api_logger.debug("Отладка API")           # НЕ выводится
api_logger.warning("Предупреждение API")  # выводится
1.4. Propagate (распространение сообщений)
python
import logging

# Настройка
root_logger = logging.getLogger()
root_logger.setLevel(logging.INFO)
root_handler = logging.StreamHandler()
root_handler.setFormatter(logging.Formatter('ROOT: %(message)s'))
root_logger.addHandler(root_handler)

# Дочерний логгер со своим обработчиком
child_logger = logging.getLogger('child')
child_logger.setLevel(logging.DEBUG)
child_handler = logging.StreamHandler()
child_handler.setFormatter(logging.Formatter('CHILD: %(message)s'))
child_logger.addHandler(child_handler)

# По умолчанию propagate=True — сообщение идёт в оба обработчика
print("=== propagate=True (по умолчанию) ===")
child_logger.info("Тестовое сообщение")

# Отключаем propagate
child_logger.propagate = False
print("\n=== propagate=False ===")
child_logger.info("Тестовое сообщение")
2. Фильтры в логировании
2.1. Что такое фильтры
Фильтры позволяют гибко управлять тем, какие сообщения попадают в обработчики.

python
import logging


class LevelFilter(logging.Filter):
    """Фильтр, пропускающий сообщения только указанного уровня."""
    
    def __init__(self, level):
        self.level = level
    
    def filter(self, record):
        return record.levelno == self.level


class ModuleFilter(logging.Filter):
    """Фильтр, пропускающий сообщения из указанного модуля."""
    
    def __init__(self, module_name):
        self.module_name = module_name
    
    def filter(self, record):
        return record.name.startswith(self.module_name)


class ContextFilter(logging.Filter):
    """Фильтр, добавляющий контекстную информацию."""
    
    def filter(self, record):
        # Добавляем пользовательские атрибуты
        record.user_id = getattr(record, 'user_id', 'anonymous')
        record.session_id = getattr(record, 'session_id', '-')
        return True
2.2. Использование фильтров
python
import logging


# Создание фильтров
info_filter = LevelFilter(logging.INFO)
error_filter = LevelFilter(logging.ERROR)
db_filter = ModuleFilter('app.database')

# Создание обработчиков
console = logging.StreamHandler()
file_handler = logging.FileHandler('app.log')

# Добавление фильтров к обработчикам
console.addFilter(info_filter)     # в консоль только INFO
file_handler.addFilter(error_filter)  # в файл только ERROR
file_handler.addFilter(db_filter)     # в файл только из БД

# Настройка логгеров
logger = logging.getLogger('app')
logger.addHandler(console)
logger.addHandler(file_handler)
logger.setLevel(logging.DEBUG)

# Использование
logger.info("Информационное сообщение")      # пойдёт в консоль
logger.error("Ошибка")                       # пойдёт в файл
logger.debug("Отладка")                      # не пойдёт никуда
2.3. Динамические фильтры
python
import logging
import os


class EnvironmentFilter(logging.Filter):
    """Фильтр, зависящий от окружения."""
    
    def __init__(self):
        self.env = os.getenv('APP_ENV', 'development')
    
    def filter(self, record):
        if self.env == 'production':
            # В продакшене только ERROR и выше
            return record.levelno >= logging.ERROR
        elif self.env == 'staging':
            # В стейджинге WARNING и выше
            return record.levelno >= logging.WARNING
        else:
            # В разработке всё
            return True


class RateLimitFilter(logging.Filter):
    """Фильтр для ограничения частоты сообщений."""
    
    def __init__(self, max_per_second=10):
        import time
        self.max_per_second = max_per_second
        self.timestamps = []
    
    def filter(self, record):
        now = time.time()
        # Удаляем старые записи
        self.timestamps = [t for t in self.timestamps if now - t < 1]
        
        if len(self.timestamps) < self.max_per_second:
            self.timestamps.append(now)
            return True
        return False
3. Продвинутая настройка обработчиков
3.1. Несколько обработчиков с разными форматами
python
import logging
from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler


def setup_advanced_logging():
    """Продвинутая настройка логирования."""
    
    # Корневой логгер
    root_logger = logging.getLogger()
    root_logger.setLevel(logging.DEBUG)
    
    # 1. Консольный обработчик (только INFO и выше)
    console = logging.StreamHandler()
    console.setLevel(logging.INFO)
    console.setFormatter(logging.Formatter(
        '%(asctime)s - %(levelname)s - %(message)s',
        datefmt='%H:%M:%S'
    ))
    root_logger.addHandler(console)
    
    # 2. Файловый обработчик (все сообщения, с ротацией по размеру)
    file_all = RotatingFileHandler(
        'logs/app.log',
        maxBytes=10_000_000,  # 10 MB
        backupCount=5,
        encoding='utf-8'
    )
    file_all.setLevel(logging.DEBUG)
    file_all.setFormatter(logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(module)s:%(lineno)d - %(message)s'
    ))
    root_logger.addHandler(file_all)
    
    # 3. Файл для ошибок (только ERROR и CRITICAL)
    file_error = RotatingFileHandler(
        'logs/errors.log',
        maxBytes=5_000_000,
        backupCount=3,
        encoding='utf-8'
    )
    file_error.setLevel(logging.ERROR)
    file_error.setFormatter(logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s\n%(exc_info)s'
    ))
    root_logger.addHandler(file_error)
    
    # 4. Ежедневная ротация для важных логов
    file_daily = TimedRotatingFileHandler(
        'logs/daily.log',
        when='midnight',
        interval=1,
        backupCount=30,
        encoding='utf-8'
    )
    file_daily.setLevel(logging.INFO)
    file_daily.setFormatter(logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    ))
    # Добавляем фильтр для важных сообщений
    file_daily.addFilter(ModuleFilter('app.important'))
    root_logger.addHandler(file_daily)
    
    return root_logger
3.2. Асинхронное логирование
python
import logging
import queue
from logging.handlers import QueueHandler, QueueListener
import threading
import time


class AsyncLogger:
    """Асинхронный логгер для повышения производительности."""
    
    def __init__(self):
        self.log_queue = queue.Queue(-1)
        self.queue_handler = QueueHandler(self.log_queue)
        
        # Настройка обработчиков
        self.console_handler = logging.StreamHandler()
        self.console_handler.setLevel(logging.INFO)
        
        self.file_handler = logging.FileHandler('async.log', encoding='utf-8')
        self.file_handler.setLevel(logging.DEBUG)
        
        # Форматтеры
        formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
        self.console_handler.setFormatter(formatter)
        self.file_handler.setFormatter(formatter)
        
        # QueueListener для обработки сообщений в фоновом потоке
        self.listener = QueueListener(
            self.log_queue,
            self.console_handler,
            self.file_handler,
            respect_handler_level=True
        )
        
        # Корневой логгер
        self.logger = logging.getLogger('async')
        self.logger.addHandler(self.queue_handler)
        self.logger.setLevel(logging.DEBUG)
    
    def start(self):
        """Запуск фоновой обработки логов."""
        self.listener.start()
    
    def stop(self):
        """Остановка фоновой обработки логов."""
        self.listener.stop()
    
    def get_logger(self):
        return self.logger


# Использование
async_logger = AsyncLogger()
async_logger.start()
logger = async_logger.get_logger()

for i in range(100):
    logger.info(f"Сообщение {i}")

async_logger.stop()
4. Кастомные форматеры
4.1. Цветной вывод в консоль
python
import logging


class ColoredFormatter(logging.Formatter):
    """Форматтер с цветным выводом в консоль."""
    
    COLORS = {
        'DEBUG': '\033[36m',      # Голубой
        'INFO': '\033[32m',       # Зелёный
        'WARNING': '\033[33m',    # Жёлтый
        'ERROR': '\033[31m',      # Красный
        'CRITICAL': '\033[35m',   # Фиолетовый
        'RESET': '\033[0m'
    }
    
    def format(self, record):
        levelname = record.levelname
        color = self.COLORS.get(levelname, self.COLORS['RESET'])
        record.levelname = f"{color}{levelname}{self.COLORS['RESET']}"
        return super().format(record)


# Использование
console = logging.StreamHandler()
console.setFormatter(ColoredFormatter(
    '%(asctime)s - %(levelname)s - %(message)s',
    datefmt='%H:%M:%S'
))
4.2. JSON форматтер
python
import logging
import json
from datetime import datetime


class JsonFormatter(logging.Formatter):
    """Форматтер для вывода логов в JSON."""
    
    def format(self, record):
        log_entry = {
            'timestamp': datetime.fromtimestamp(record.created).isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno,
            'message': record.getMessage(),
            'process': record.process,
            'thread': record.thread
        }
        
        # Добавляем исключение, если есть
        if record.exc_info:
            log_entry['exception'] = self.formatException(record.exc_info)
        
        # Добавляем дополнительные атрибуты
        for key, value in record.__dict__.items():
            if key not in log_entry and not key.startswith('_'):
                log_entry[key] = str(value)
        
        return json.dumps(log_entry, ensure_ascii=False)


# Использование
json_handler = logging.FileHandler('logs.json', encoding='utf-8')
json_handler.setFormatter(JsonFormatter())
4.3. Форматтер с контекстной информацией
python
import logging
import uuid


class ContextFormatter(logging.Formatter):
    """Форматтер, добавляющий контекст выполнения."""
    
    def format(self, record):
        # Добавляем request_id, если нет
        if not hasattr(record, 'request_id'):
            record.request_id = getattr(record, 'request_id', '-')
        
        # Добавляем user_id
        if not hasattr(record, 'user_id'):
            record.user_id = getattr(record, 'user_id', '-')
        
        # Добавляем execution_time
        if hasattr(record, 'execution_time'):
            record.execution_time = f"{record.execution_time:.3f}s"
        
        return super().format(record)


# Контекстный менеджер для логирования времени выполнения
import time
from contextlib import contextmanager

@contextmanager
def log_execution(logger, operation: str, **context):
    """Логирование времени выполнения операции."""
    start = time.time()
    try:
        yield
        elapsed = time.time() - start
        logger.info(
            f"Operation '{operation}' completed",
            extra={
                'execution_time': elapsed,
                **context
            }
        )
    except Exception as e:
        elapsed = time.time() - start
        logger.error(
            f"Operation '{operation}' failed: {e}",
            extra={
                'execution_time': elapsed,
                'error': str(e),
                **context
            }
        )
        raise


# Настройка
logger = logging.getLogger('app')
handler = logging.StreamHandler()
handler.setFormatter(ContextFormatter(
    '%(asctime)s - [%(request_id)s] - %(levelname)s - %(message)s - exec_time=%(execution_time)s'
))
logger.addHandler(handler)
5. Логирование в разных окружениях
5.1. Конфигурация через словарь
python
import logging.config
import os


LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    
    'formatters': {
        'simple': {
            'format': '%(asctime)s - %(levelname)s - %(message)s',
            'datefmt': '%H:%M:%S'
        },
        'detailed': {
            'format': '%(asctime)s - %(name)s - %(levelname)s - %(module)s:%(lineno)d - %(message)s'
        },
        'json': {
            '()': 'utils.JsonFormatter',
        },
    },
    
    'filters': {
        'production_filter': {
            '()': 'utils.EnvironmentFilter',
        },
    },
    
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'INFO',
            'formatter': 'simple',
            'stream': 'ext://sys.stdout'
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'level': 'DEBUG',
            'formatter': 'detailed',
            'filename': 'logs/app.log',
            'maxBytes': 10_000_000,
            'backupCount': 5,
            'encoding': 'utf-8'
        },
        'error_file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'level': 'ERROR',
            'formatter': 'detailed',
            'filename': 'logs/errors.log',
            'maxBytes': 5_000_000,
            'backupCount': 3,
            'encoding': 'utf-8'
        },
    },
    
    'loggers': {
        'app': {
            'level': 'DEBUG',
            'handlers': ['console', 'file'],
            'propagate': False
        },
        'app.database': {
            'level': 'INFO',
            'handlers': ['file'],
            'propagate': True
        },
        'app.api': {
            'level': 'WARNING',
            'handlers': ['console', 'error_file'],
            'propagate': False
        },
    },
    
    'root': {
        'level': 'WARNING',
        'handlers': ['console']
    }
}


def setup_logging(env: str = 'development'):
    """Настройка логирования в зависимости от окружения."""
    
    if env == 'production':
        # В продакшене — только файлы, JSON формат
        LOGGING_CONFIG['handlers']['console']['level'] = 'ERROR'
        LOGGING_CONFIG['handlers']['file']['formatter'] = 'json'
    elif env == 'development':
        # В разработке — подробные логи в консоль
        LOGGING_CONFIG['handlers']['console']['level'] = 'DEBUG'
    
    logging.config.dictConfig(LOGGING_CONFIG)


# Использование
setup_logging(os.getenv('APP_ENV', 'development'))
logger = logging.getLogger('app')
5.2. Логирование в Docker/Cloud
python
import logging
import sys
import json


class CloudLoggingFormatter(logging.Formatter):
    """Форматтер для облачных окружений (JSON Lines)."""
    
    def format(self, record):
        log_entry = {
            'timestamp': self.formatTime(record),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'service': os.getenv('SERVICE_NAME', 'unknown'),
            'instance': os.getenv('HOSTNAME', 'unknown')
        }
        
        if record.exc_info:
            log_entry['exception'] = self.formatException(record.exc_info)
        
        return json.dumps(log_entry)


def setup_cloud_logging():
    """Настройка логирования для облачных окружений."""
    
    # В stdout (для сбора Docker/kubernetes)
    stdout_handler = logging.StreamHandler(sys.stdout)
    stdout_handler.setFormatter(CloudLoggingFormatter())
    
    # В stderr для ошибок
    stderr_handler = logging.StreamHandler(sys.stderr)
    stderr_handler.setLevel(logging.ERROR)
    stderr_handler.setFormatter(CloudLoggingFormatter())
    
    root_logger = logging.getLogger()
    root_logger.setLevel(logging.INFO)
    root_logger.addHandler(stdout_handler)
    root_logger.addHandler(stderr_handler)
6. Практические примеры
6.1. Полная настройка для веб-приложения
python
# logging_config.py
import logging
import logging.config
from pathlib import Path
import sys


def setup_web_app_logging():
    """Настройка логирования для веб-приложения."""
    
    # Создание директорий для логов
    Path("logs").mkdir(exist_ok=True)
    Path("logs/access").mkdir(exist_ok=True)
    Path("logs/error").mkdir(exist_ok=True)
    
    config = {
        'version': 1,
        'disable_existing_loggers': False,
        
        'formatters': {
            'access': {
                'format': '%(asctime)s - %(remote_addr)s - %(request_method)s %(request_path)s - %(status_code)s - %(duration).3fs',
                'datefmt': '%Y-%m-%d %H:%M:%S'
            },
            'error': {
                'format': '%(asctime)s - %(levelname)s - %(name)s - %(message)s\n%(exc_info)s',
                'datefmt': '%Y-%m-%d %H:%M:%S'
            },
            'app': {
                'format': '%(asctime)s - %(levelname)s - [%(request_id)s] - %(name)s:%(lineno)d - %(message)s'
            }
        },
        
        'filters': {
            'request_filter': {
                '()': 'app.filters.RequestFilter'
            }
        },
        
        'handlers': {
            'console': {
                'class': 'logging.StreamHandler',
                'level': 'INFO',
                'formatter': 'app',
                'stream': sys.stdout
            },
            'access_file': {
                'class': 'logging.handlers.TimedRotatingFileHandler',
                'level': 'INFO',
                'formatter': 'access',
                'filename': 'logs/access/access.log',
                'when': 'midnight',
                'backupCount': 30,
                'encoding': 'utf-8'
            },
            'error_file': {
                'class': 'logging.handlers.TimedRotatingFileHandler',
                'level': 'WARNING',
                'formatter': 'error',
                'filename': 'logs/error/error.log',
                'when': 'midnight',
                'backupCount': 30,
                'encoding': 'utf-8'
            },
            'app_file': {
                'class': 'logging.handlers.RotatingFileHandler',
                'level': 'DEBUG',
                'formatter': 'app',
                'filename': 'logs/app.log',
                'maxBytes': 10_000_000,
                'backupCount': 5,
                'encoding': 'utf-8'
            }
        },
        
        'loggers': {
            'app': {
                'level': 'DEBUG',
                'handlers': ['console', 'app_file'],
                'propagate': False
            },
            'app.access': {
                'level': 'INFO',
                'handlers': ['access_file'],
                'propagate': False
            },
            'app.error': {
                'level': 'WARNING',
                'handlers': ['error_file', 'console'],
                'propagate': False
            },
            'sqlalchemy': {
                'level': 'WARNING',
                'handlers': ['app_file'],
                'propagate': False
            }
        },
        
        'root': {
            'level': 'WARNING',
            'handlers': ['console']
        }
    }
    
    logging.config.dictConfig(config)
7. Контрольные вопросы
Как организована иерархия логгеров в Python?

Что означает параметр propagate у логгера?

Как создать фильтр, пропускающий сообщения только определённого уровня?

В чём разница между Filter и Handler?

Как добавить пользовательские атрибуты в логи?

Как настроить несколько обработчиков с разными уровнями?

Как реализовать цветной вывод в консоль?

Что такое QueueHandler и когда он используется?

Как настроить логирование для разных окружений (dev/prod)?

Как добавить контекстную информацию (request_id, user_id) в логи?

8. Практическое задание
Задание 1 (базовое)
Создайте иерархию логгеров для приложения с модулями auth, database, api. Настройте разные уровни логирования для каждого.

Задание 2 (среднее)
Создайте фильтр, который записывает в отдельный файл только сообщения об ошибках из модуля database.

Задание 3 (сложное)
Настройте продвинутое логирование для веб-приложения:

Access-логи (все запросы)

Error-логи (только ошибки)

Application-логи (с контекстом)

Ротация по времени и размеру

9. Шпаргалка
python
# === ИЕРАРХИЯ ЛОГГЕРОВ ===
logger = logging.getLogger('app.module.submodule')

# === ФИЛЬТР ===
class MyFilter(logging.Filter):
    def filter(self, record):
        return record.levelno >= logging.WARNING

# === НЕСКОЛЬКО ОБРАБОТЧИКОВ ===
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# === КАСТОМНЫЙ ФОРМАТТЕР ===
class CustomFormatter(logging.Formatter):
    def format(self, record):
        return f"{record.levelname}: {record.getMessage()}"

# === DICT CONFIG ===
logging.config.dictConfig({
    'version': 1,
    'handlers': {...},
    'loggers': {...}
})
Итог лекции
Вы сегодня:

✅ Изучили иерархию логгеров и принцип распространения сообщений

✅ Освоили создание фильтров для гибкой маршрутизации логов

✅ Научились настраивать несколько обработчиков с разными форматами

✅ Создали кастомные форматеры (цветной вывод, JSON)

✅ Настроили логирование для разных окружений

Теперь вы можете создавать профессиональные системы логирования для любых проектов!
