 Тема 2.13. Логирование в Python: модуль logging. Уровни, форматирование, ротация

**Цель лекции:**  
Научиться профессионально логировать Python-приложения: настраивать модуль `logging`, использовать различные уровни логирования, форматировать вывод, организовывать ротацию лог-файлов.

> Главная мысль: **Логирование — это чёрный ящик самолёта для программиста. Без него вы никогда не узнаете, что произошло на самом деле.**

---

## Содержание

1. [Введение в логирование](#1-введение-в-логирование)
2. [Модуль logging: основы](#2-модуль-logging-основы)
3. [Уровни логирования](#3-уровни-логирования)
4. [Логгеры, обработчики, форматтеры](#4-логгеры-обработчики-форматтеры)
5. [Форматирование логов](#5-форматирование-логов)
6. [Ротация лог-файлов](#6-ротация-лог-файлов)
7. [Настройка через конфигурационный файл](#7-настройка-через-конфигурационный-файл)
8. [Логирование в разных модулях](#8-логирование-в-разных-модулях)
9. [Практические примеры](#9-практические-примеры)
10. [Контрольные вопросы](#10-контрольные-вопросы)
11. [Практическое задание](#11-практическое-задание)
12. [Шпаргалка](#12-шпаргалка)

---

## 1. Введение в логирование

### 1.1. Что такое логирование и зачем оно нужно

**Логирование** — это процесс записи информации о работе программы.

**Чем логирование лучше `print()`?**

| `print()` | `logging` |
|-----------|-----------|
| Нельзя отключить | Можно отключить по уровням |
| Нет уровней важности | 6 уровней (DEBUG, INFO, WARNING...) |
| Вывод только в консоль | Вывод в файл, консоль, базу данных, email |
| Нет информации о времени | Автоматическая временная метка |
| Нельзя настроить формат | Гибкое форматирование |
| Трудно анализировать | Поддержка ротации, фильтрации |

### 1.2. Пирамида логирования
┌─────────────┐
│ ERROR │ ← Только критические ошибки
│ WARNING │ ← Потенциальные проблемы
│ INFO │ ← Ход выполнения программы
│ DEBUG │ ← Детальная диагностика
└─────────────┘
Чем ниже уровень, тем больше логов

text

---

## 2. Модуль logging: основы

### 2.1. Простейшее логирование

```python
import logging

# Базовая настройка
logging.basicConfig(level=logging.DEBUG)

# Логирование сообщений
logging.debug("Отладочное сообщение")
logging.info("Информационное сообщение")
logging.warning("Предупреждение")
logging.error("Ошибка")
logging.critical("Критическая ошибка")
Вывод:

text
DEBUG:root:Отладочное сообщение
INFO:root:Информационное сообщение
WARNING:root:Предупреждение
ERROR:root:Ошибка
CRITICAL:root:Критическая ошибка
2.2. Настройка формата вывода
python
import logging

# Настройка формата
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

logging.info("Приложение запущено")
Вывод:

text
2024-01-15 14:30:25 - INFO - Приложение запущено
2.3. Вывод в файл
python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),     # Запись в файл
        logging.StreamHandler()              # Вывод в консоль
    ]
)

logging.info("Сообщение будет и в файле, и в консоли")
3. Уровни логирования
3.1. Шкала уровней
Уровень	Числовое значение	Когда использовать
DEBUG	10	Детальная информация для диагностики
INFO	20	Подтверждение корректной работы
WARNING	30	Неожиданное событие, но программа работает
ERROR	40	Ошибка, но программа продолжает работу
CRITICAL	50	Критическая ошибка, программа может остановиться
3.2. Установка порога
python
import logging

# Устанавливаем порог INFO (DEBUG и ниже игнорируются)
logging.basicConfig(level=logging.INFO)

logging.debug("Это сообщение НЕ будет показано")    # Игнорируется
logging.info("Это сообщение будет показано")        # Показывается
logging.warning("Это сообщение будет показано")     # Показывается
3.3. Динамическое изменение уровня
python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# В зависимости от режима работы
import os
if os.getenv("DEBUG"):
    logger.setLevel(logging.DEBUG)
else:
    logger.setLevel(logging.INFO)
4. Логгеры, обработчики, форматтеры
4.1. Архитектура модуля logging
text
┌─────────────────────────────────────────────────────────┐
│                      Логгер (Logger)                     │
│                 (создаёт сообщения)                      │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                   Фильтр (Filter)                        │
│              (пропускает/блокирует)                      │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                 Обработчик (Handler)                     │
│              (определяет куда выводить)                  │
└────────┬──────────────────────┬─────────────────────────┘
         │                      │
         ▼                      ▼
┌─────────────────┐    ┌─────────────────┐
│  FileHandler    │    │ StreamHandler   │
│  (в файл)       │    │  (в консоль)    │
└────────┬────────┘    └────────┬────────┘
         │                      │
         ▼                      ▼
┌─────────────────────────────────────────────────────────┐
│                  Форматтер (Formatter)                   │
│              (определяет формат вывода)                  │
└─────────────────────────────────────────────────────────┘
4.2. Создание логгера
python
import logging

# Создание логгера
logger = logging.getLogger('my_app')
logger.setLevel(logging.DEBUG)

# Создание обработчика для файла
file_handler = logging.FileHandler('app.log')
file_handler.setLevel(logging.INFO)

# Создание обработчика для консоли
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.DEBUG)

# Создание форматтера
formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

# Привязка форматтера к обработчикам
file_handler.setFormatter(formatter)
console_handler.setFormatter(formatter)

# Привязка обработчиков к логгеру
logger.addHandler(file_handler)
logger.addHandler(console_handler)

# Использование
logger.debug("Отладка")
logger.info("Информация")
logger.warning("Предупреждение")
4.3. Типы обработчиков (handlers)
Обработчик	Назначение
StreamHandler	Вывод в консоль (stdout/stderr)
FileHandler	Запись в файл
RotatingFileHandler	Файл с ротацией по размеру
TimedRotatingFileHandler	Файл с ротацией по времени
SMTPHandler	Отправка по email
HTTPHandler	Отправка на HTTP сервер
SocketHandler	Отправка через сокет
SysLogHandler	Отправка в syslog
5. Форматирование логов
5.1. Атрибуты форматирования
Атрибут	Описание	Пример
%(asctime)s	Время события	2024-01-15 14:30:25
%(levelname)s	Уровень логирования	INFO
%(levelno)s	Числовой код уровня	20
%(name)s	Имя логгера	my_app
%(message)s	Сообщение	User logged in
%(filename)s	Имя файла	main.py
%(lineno)d	Номер строки	42
%(funcName)s	Имя функции	login
%(module)s	Имя модуля	main
%(pathname)s	Полный путь к файлу	/app/main.py
%(process)d	PID процесса	12345
%(thread)d	ID потока	123456
5.2. Примеры форматов
python
# Простой формат
format='%(levelname)s: %(message)s'

# С временем
format='%(asctime)s - %(levelname)s - %(message)s'

# С именем модуля и функцией
format='%(asctime)s - %(module)s:%(lineno)d - %(funcName)s - %(levelname)s - %(message)s'

# JSON формат (для машинного чтения)
import json
class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_entry = {
            'timestamp': self.formatTime(record),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'module': record.module,
            'line': record.lineno
        }
        return json.dumps(log_entry)
5.3. Цветной вывод в консоль
python
import logging

class ColoredFormatter(logging.Formatter):
    """Цветное форматирование для консоли."""
    
    COLORS = {
        'DEBUG': '\033[36m',      # Голубой
        'INFO': '\033[32m',       # Зелёный
        'WARNING': '\033[33m',    # Жёлтый
        'ERROR': '\033[31m',      # Красный
        'CRITICAL': '\033[35m',   # Фиолетовый
    }
    RESET = '\033[0m'
    
    def format(self, record):
        color = self.COLORS.get(record.levelname, self.RESET)
        record.levelname = f"{color}{record.levelname}{self.RESET}"
        return super().format(record)

# Использование
console_handler = logging.StreamHandler()
console_handler.setFormatter(ColoredFormatter('%(asctime)s - %(levelname)s - %(message)s'))
6. Ротация лог-файлов
6.1. RotatingFileHandler (ротация по размеру)
python
import logging
from logging.handlers import RotatingFileHandler

# Создание обработчика с ротацией
handler = RotatingFileHandler(
    'app.log',           # имя файла
    maxBytes=1048576,    # максимальный размер 1 MB (1024 * 1024)
    backupCount=5        # хранить 5 резервных копий
)

handler.setLevel(logging.DEBUG)
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

logger = logging.getLogger(__name__)
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

# При достижении размера 1 MB, файлы будут переименованы:
# app.log → app.log.1 → app.log.2 → ... → app.log.5
for i in range(10000):
    logger.info(f"Сообщение номер {i}")
6.2. TimedRotatingFileHandler (ротация по времени)
python
import logging
from logging.handlers import TimedRotatingFileHandler

# Создание обработчика с ротацией по времени
handler = TimedRotatingFileHandler(
    'app.log',           # имя файла
    when='midnight',     # ротация в полночь
    interval=1,          # каждый день
    backupCount=30       # хранить 30 дней
)

# Другие значения параметра when:
# 'S' - секунды
# 'M' - минуты
# 'H' - часы
# 'D' - дни
# 'W0'-'W6' - дни недели (понедельник-воскресенье)
# 'midnight' - в полночь

handler.setLevel(logging.DEBUG)
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

logger = logging.getLogger(__name__)
logger.addHandler(handler)
6.3. Пример ротации с перезаписью
python
import logging
from logging.handlers import RotatingFileHandler
from pathlib import Path

def setup_logger(name: str, log_dir: str = "logs", max_bytes: int = 1_000_000, backup_count: int = 5):
    """
    Настройка логгера с ротацией.
    
    Args:
        name: Имя логгера
        log_dir: Папка для логов
        max_bytes: Максимальный размер файла в байтах
        backup_count: Количество резервных копий
    """
    # Создаём папку для логов
    Path(log_dir).mkdir(exist_ok=True)
    
    # Создаём логгер
    logger = logging.getLogger(name)
    logger.setLevel(logging.DEBUG)
    
    # Удаляем старые обработчики (чтобы не дублировать)
    if logger.handlers:
        logger.handlers.clear()
    
    # Обработчик для файлов (с ротацией)
    file_handler = RotatingFileHandler(
        f"{log_dir}/{name}.log",
        maxBytes=max_bytes,
        backupCount=backup_count,
        encoding='utf-8'
    )
    file_handler.setLevel(logging.DEBUG)
    
    # Обработчик для консоли (цветной)
    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.INFO)
    
    # Форматтер
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(module)s:%(lineno)d - %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )
    
    file_handler.setFormatter(formatter)
    console_handler.setFormatter(formatter)
    
    logger.addHandler(file_handler)
    logger.addHandler(console_handler)
    
    return logger

# Использование
logger = setup_logger('my_app')
logger.info("Приложение запущено")
7. Настройка через конфигурационный файл
7.1. Файл logging.conf
ini
[loggers]
keys=root,app

[handlers]
keys=consoleHandler,fileHandler

[formatters]
keys=simpleFormatter,detailedFormatter

[logger_root]
level=WARNING
handlers=consoleHandler

[logger_app]
level=DEBUG
handlers=consoleHandler,fileHandler
qualname=app
propagate=0

[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=simpleFormatter
args=(sys.stdout,)

[handler_fileHandler]
class=logging.handlers.RotatingFileHandler
level=DEBUG
formatter=detailedFormatter
args=('app.log', 'a', 1048576, 5)

[formatter_simpleFormatter]
format=%(asctime)s - %(levelname)s - %(message)s

[formatter_detailedFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(module)s:%(lineno)d - %(message)s
datefmt=%Y-%m-%d %H:%M:%S
7.2. Загрузка конфигурации
python
import logging
import logging.config

# Загрузка из файла
logging.config.fileConfig('logging.conf')

# Использование
logger = logging.getLogger('app')
logger.debug("Отладочное сообщение")
logger.info("Информационное сообщение")
7.3. Конфигурация через словарь (рекомендуемый способ)
python
import logging
import logging.config

LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'simple': {
            'format': '%(asctime)s - %(levelname)s - %(message)s',
            'datefmt': '%H:%M:%S'
        },
        'detailed': {
            'format': '%(asctime)s - %(name)s - %(levelname)s - %(module)s:%(lineno)d - %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S'
        },
        'json': {
            '()': 'my_module.JSONFormatter',
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
            'filename': 'app.log',
            'maxBytes': 1048576,
            'backupCount': 5,
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
    },
    'root': {
        'level': 'WARNING',
        'handlers': ['console']
    }
}

# Загрузка конфигурации
logging.config.dictConfig(LOGGING_CONFIG)

# Использование
logger = logging.getLogger('app')
logger.info("Приложение запущено")
8. Логирование в разных модулях
8.1. Иерархия логгеров
python
# main.py
import logging

logger = logging.getLogger(__name__)  # __name__ = "__main__"

def main():
    logger.info("Главная функция запущена")
    # ...

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    main()
python
# database.py
import logging

logger = logging.getLogger(__name__)  # __name__ = "database"

def connect():
    logger.info("Подключение к базе данных")
    # ...
8.2. Использование name
python
# utils/logger.py
import logging
import sys

def setup_logger(level=logging.INFO):
    """Настройка корневого логгера."""
    logging.basicConfig(
        level=level,
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        handlers=[
            logging.StreamHandler(sys.stdout),
            logging.FileHandler('app.log', encoding='utf-8')
        ]
    )

def get_logger(name):
    """Получение логгера с именем модуля."""
    return logging.getLogger(name)
python
# user_service.py
from utils.logger import get_logger

logger = get_logger(__name__)

class UserService:
    def create_user(self, email):
        logger.info(f"Создание пользователя: {email}")
        try:
            # логика создания
            pass
        except Exception as e:
            logger.error(f"Ошибка создания пользователя {email}: {e}", exc_info=True)
            raise
8.3. Логирование исключений
python
import logging

logger = logging.getLogger(__name__)

try:
    result = 10 / 0
except ZeroDivisionError as e:
    # exc_info=True добавляет стек вызовов
    logger.error("Произошла ошибка", exc_info=True)
    
    # или с сообщением
    logger.exception("Ошибка при делении")  # автоматически exc_info=True
9. Практические примеры
9.1. Пример: веб-приложение
python
import logging
import time
from functools import wraps
from datetime import datetime

# Настройка логгера
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger('web_app')


def log_request(func):
    """Декоратор для логирования запросов."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        logger.info(f"Вызов {func.__name__} с аргументами: {args}, {kwargs}")
        
        try:
            result = func(*args, **kwargs)
            elapsed = time.time() - start_time
            logger.info(f"{func.__name__} выполнен за {elapsed:.3f} сек")
            return result
        except Exception as e:
            logger.error(f"Ошибка в {func.__name__}: {e}", exc_info=True)
            raise
    
    return wrapper


class UserAPI:
    @log_request
    def get_user(self, user_id: int):
        logger.debug(f"Поиск пользователя с id={user_id}")
        if user_id < 0:
            logger.warning(f"Попытка поиска с отрицательным id: {user_id}")
            return None
        # логика получения пользователя
        return {"id": user_id, "name": "Alice"}
    
    @log_request
    def create_user(self, data: dict):
        logger.info(f"Создание пользователя: {data.get('email')}")
        # логика создания
        return {"id": 1, **data}


# Использование
api = UserAPI()
user = api.get_user(42)
api.create_user({"email": "test@example.com", "name": "Test"})
9.2. Пример: логирование в базу данных
python
import logging
import sqlite3
from datetime import datetime

class DatabaseHandler(logging.Handler):
    """Обработчик для записи логов в базу данных."""
    
    def __init__(self, db_path: str):
        super().__init__()
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self):
        with sqlite3.connect(self.db_path) as conn:
            conn.execute("""
                CREATE TABLE IF NOT EXISTS logs (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    timestamp TEXT,
                    level TEXT,
                    logger TEXT,
                    module TEXT,
                    message TEXT
                )
            """)
    
    def emit(self, record):
        log_entry = {
            'timestamp': datetime.now().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'module': record.module,
            'message': self.format(record)
        }
        
        with sqlite3.connect(self.db_path) as conn:
            conn.execute("""
                INSERT INTO logs (timestamp, level, logger, module, message)
                VALUES (?, ?, ?, ?, ?)
            """, (
                log_entry['timestamp'],
                log_entry['level'],
                log_entry['logger'],
                log_entry['module'],
                log_entry['message']
            ))


# Использование
db_handler = DatabaseHandler('logs.db')
db_handler.setFormatter(logging.Formatter('%(asctime)s - %(message)s'))

logger = logging.getLogger(__name__)
logger.addHandler(db_handler)
logger.setLevel(logging.INFO)

logger.info("Это сообщение попадёт в базу данных")
9.3. Пример: логирование с разными уровнями
python
import logging

class Application:
    def __init__(self, debug_mode=False):
        self._setup_logging(debug_mode)
    
    def _setup_logging(self, debug_mode):
        # Консольный обработчик
        console = logging.StreamHandler()
        console.setLevel(logging.DEBUG if debug_mode else logging.INFO)
        
        # Файловый обработчик (только ошибки и выше)
        file_handler = logging.FileHandler('errors.log')
        file_handler.setLevel(logging.ERROR)
        
        # Форматтер
        formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s'
        )
        console.setFormatter(formatter)
        file_handler.setFormatter(formatter)
        
        # Корневой логгер
        root_logger = logging.getLogger()
        root_logger.setLevel(logging.DEBUG if debug_mode else logging.INFO)
        root_logger.addHandler(console)
        root_logger.addHandler(file_handler)
        
        self.logger = logging.getLogger(__name__)
    
    def run(self):
        self.logger.debug("Запуск приложения (отладка)")
        self.logger.info("Приложение запущено")
        
        try:
            self._do_work()
        except Exception as e:
            self.logger.critical(f"Критическая ошибка: {e}", exc_info=True)
            raise
    
    def _do_work(self):
        self.logger.info("Выполнение работы")
        # имитация работы
        import random
        if random.random() < 0.3:
            raise ValueError("Случайная ошибка")


if __name__ == "__main__":
    app = Application(debug_mode=True)
    app.run()
10. Контрольные вопросы
Почему логирование лучше, чем print()?

Какие уровни логирования существуют и когда их использовать?

В чём разница между logger.error() и logger.exception()?

Что такое ротация логов и зачем она нужна?

Как настроить логирование в файл с ротацией по размеру?

Какие атрибуты можно использовать в форматировании логов?

Как добавить цветной вывод в консоль?

Как правильно организовать логирование в многомодульном проекте?

Что делает параметр exc_info=True?

Как динамически изменить уровень логирования во время работы программы?

11. Практическое задание
Задание 1 (базовое)
Настройте логирование для небольшого скрипта, которое пишет в файл app.log и выводит в консоль. Используйте разные уровни логирования.

Задание 2 (среднее)
Создайте класс LoggerSetup с методами:

setup_console_logging() — настройка вывода в консоль

setup_file_logging() — настройка вывода в файл с ротацией

setup_database_logging() — настройка записи в SQLite

Задание 3 (сложное)
Создайте декоратор @log_call, который логирует вызов функции:

Входные параметры

Выходные значения

Время выполнения

Исключения

12. Шпаргалка
python
# === БЫСТРЫЙ СТАРТ ===
import logging
logging.basicConfig(level=logging.INFO)
logging.info("Сообщение")

# === НАСТРОЙКА ===
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S',
    filename='app.log',
    filemode='a'  # 'a' - append, 'w' - перезапись
)

# === ПРОДВИНУТАЯ НАСТРОЙКА ===
logger = logging.getLogger(__name__)

# Обработчики
console = logging.StreamHandler()
file_handler = logging.FileHandler('app.log')
rotating = logging.handlers.RotatingFileHandler('app.log', maxBytes=1_000_000, backupCount=5)

# Форматтер
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
console.setFormatter(formatter)

# Добавление
logger.addHandler(console)
logger.setLevel(logging.DEBUG)

# === УРОВНИ ===
logger.debug("Отладка")
logger.info("Информация")
logger.warning("Предупреждение")
logger.error("Ошибка")
logger.critical("Критическая ошибка")
logger.exception("Ошибка с трейсом")  # только в except

# === РОТАЦИЯ ===
from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler

# По размеру
handler = RotatingFileHandler('app.log', maxBytes=1_000_000, backupCount=5)

# По времени
handler = TimedRotatingFileHandler('app.log', when='midnight', interval=1, backupCount=30)
Итог лекции
Вы сегодня:

✅ Изучили модуль logging и его возможности

✅ Освоили 6 уровней логирования

✅ Научились настраивать форматирование вывода

✅ Поняли архитектуру логгеров, обработчиков, форматтеров

✅ Освоили ротацию лог-файлов

✅ Научились логировать исключения

Теперь ваши приложения будут иметь профессиональное логирование!

