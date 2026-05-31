# ПЗ 2.16. Настройка системы логирования: файлы, консоль

**Тема:** Логирование в Python, модуль logging, уровни, форматирование, обработчики

**Цель работы:**  
Научиться настраивать систему логирования для Python-приложений: выводить логи в консоль и файл, настраивать уровни логирования, форматы, ротацию лог-файлов.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+

> Главная мысль: **Хорошо настроенное логирование — это половина успешной отладки.**

---

## Содержание

1. [Теоретическая справка](#1-теоретическая-справка)
2. [Нулевой вариант (эталонный)](#2-нулевой-вариант-эталонный)
3. [25 вариантов практической работы](#3-25-вариантов-практической-работы)
4. [Критерии оценки](#4-критерии-оценки)
5. [Шпаргалка](#5-шпаргалка)

---

## 1. Теоретическая справка

### 1.1. Компоненты системы логирования
┌─────────────────────────────────────────────────────────────────┐
│ Логгер (Logger) │
│ (создаёт сообщения, имеет имя) │
└───────────────────────────────┬─────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ Фильтр (Filter) │
│ (пропускает или блокирует сообщения) │
└───────────────────────────────┬─────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ Обработчик (Handler) │
│ (определяет КУДА отправлять сообщения) │
└───────────────┬───────────────────────────────┬─────────────────┘
│ │
▼ ▼
┌──────────────────────────────┐ ┌──────────────────────────────┐
│ ConsoleHandler │ │ FileHandler │
│ (вывод в консоль) │ │ (вывод в файл) │
└───────────────┬──────────────┘ └───────────────┬──────────────┘
│ │
▼ ▼
┌──────────────────────────────┐ ┌──────────────────────────────┐
│ Formatter │ │ Formatter │
│ (определяет ФОРМАТ вывода) │ │ (определяет ФОРМАТ вывода) │
└──────────────────────────────┘ └──────────────────────────────┘

text

### 1.2. Уровни логирования

| Уровень | Значение | Когда использовать |
|---------|----------|---------------------|
| `DEBUG` | 10 | Детальная диагностическая информация |
| `INFO` | 20 | Подтверждение正常工作ы программы |
| `WARNING` | 30 | Неожиданное событие, но программа работает |
| `ERROR` | 40 | Ошибка, программа не может выполнить операцию |
| `CRITICAL` | 50 | Критическая ошибка, программа может остановиться |

### 1.3. Основные обработчики (Handlers)

| Обработчик | Назначение |
|------------|------------|
| `StreamHandler` | Вывод в консоль (stdout/stderr) |
| `FileHandler` | Запись в файл |
| `RotatingFileHandler` | Запись в файл с ротацией по размеру |
| `TimedRotatingFileHandler` | Запись в файл с ротацией по времени |

### 1.4. Атрибуты форматирования

| Атрибут | Описание | Пример |
|---------|----------|--------|
| `%(asctime)s` | Время события | `2024-01-15 14:30:25` |
| `%(levelname)s` | Уровень логирования | `INFO` |
| `%(name)s` | Имя логгера | `my_app` |
| `%(message)s` | Сообщение | `User logged in` |
| `%(filename)s` | Имя файла | `main.py` |
| `%(lineno)d` | Номер строки | `42` |
| `%(funcName)s` | Имя функции | `login` |
| `%(module)s` | Имя модуля | `main` |
| `%(process)d` | PID процесса | `12345` |
| `%(thread)d` | ID потока | `123456` |

---

## 2. Нулевой вариант (эталонный)

**Назначение:** преподаватель выполняет этот вариант на занятии, показывая настройку системы логирования.

### Техническое задание (нулевой вариант)

> Разработайте модуль `logger_setup.py`, который настраивает систему логирования для приложения. Требования:
> 1. Вывод логов в консоль с цветным форматированием (INFO и выше)
> 2. Вывод логов в файл `app.log` с ротацией (размер 1 МБ, 3 резервные копии)
> 3. Разные форматы: для консоли — краткий, для файла — подробный
> 4. Возможность переключать уровень логирования через переменную окружения
> 5. Функция `get_logger(name)` для получения настроенного логгера

### Эталонная реализация

```python
#!/usr/bin/env python3
"""
logger_setup.py — Настройка системы логирования.

Использование:
    from logger_setup import get_logger
    
    logger = get_logger(__name__)
    logger.info("Сообщение")
"""

import os
import sys
import logging
from logging.handlers import RotatingFileHandler
from pathlib import Path
from typing import Optional


# ============================================================
# ЦВЕТА ДЛЯ КОНСОЛИ
# ============================================================

class Colors:
    """ANSI цвета для терминала."""
    RESET = '\033[0m'
    BOLD = '\033[1m'
    DIM = '\033[2m'
    
    # Цвета текста
    BLACK = '\033[30m'
    RED = '\033[31m'
    GREEN = '\033[32m'
    YELLOW = '\033[33m'
    BLUE = '\033[34m'
    MAGENTA = '\033[35m'
    CYAN = '\033[36m'
    WHITE = '\033[37m'
    
    # Цвета фона
    BG_RED = '\033[41m'
    BG_GREEN = '\033[42m'
    BG_YELLOW = '\033[43m'
    BG_BLUE = '\033[44m'


# Цвета для уровней логирования
LEVEL_COLORS = {
    logging.DEBUG: Colors.CYAN,
    logging.INFO: Colors.GREEN,
    logging.WARNING: Colors.YELLOW,
    logging.ERROR: Colors.RED,
    logging.CRITICAL: f"{Colors.BOLD}{Colors.RED}",
}


# ============================================================
# ФОРМАТТЕРЫ
# ============================================================

class ColoredFormatter(logging.Formatter):
    """Форматтер с цветным выводом для консоли."""
    
    def __init__(self, fmt: str, datefmt: str = None):
        super().__init__(fmt, datefmt)
    
    def format(self, record: logging.LogRecord) -> str:
        # Сохраняем оригинальный уровень
        original_levelname = record.levelname
        
        # Добавляем цвет
        color = LEVEL_COLORS.get(record.levelno, Colors.RESET)
        record.levelname = f"{color}{record.levelname}{Colors.RESET}"
        
        # Форматируем сообщение
        result = super().format(record)
        
        # Восстанавливаем оригинальный уровень
        record.levelname = original_levelname
        
        return result


class JsonFormatter(logging.Formatter):
    """Форматтер для вывода в JSON (для машинного чтения)."""
    
    def format(self, record: logging.LogRecord) -> str:
        import json
        from datetime import datetime
        
        log_entry = {
            'timestamp': datetime.now().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno,
            'message': record.getMessage(),
            'process': record.process,
            'thread': record.thread
        }
        
        # Добавляем информацию об исключении, если есть
        if record.exc_info:
            log_entry['exception'] = self.formatException(record.exc_info)
        
        return json.dumps(log_entry, ensure_ascii=False)


# ============================================================
# ОСНОВНОЙ КЛАСС НАСТРОЙКИ
# ============================================================

class LoggingSetup:
    """
    Класс для настройки системы логирования.
    
    Пример использования:
        setup = LoggingSetup()
        setup.configure()
        logger = setup.get_logger(__name__)
    """
    
    def __init__(
        self,
        log_dir: str = "logs",
        app_name: str = "app",
        console_level: str = "INFO",
        file_level: str = "DEBUG",
        max_bytes: int = 1_048_576,  # 1 MB
        backup_count: int = 3,
        use_json: bool = False
    ):
        """
        Инициализация настроек логирования.
        
        Args:
            log_dir: Папка для хранения лог-файлов
            app_name: Имя приложения (используется в имени файла)
            console_level: Уровень логирования для консоли
            file_level: Уровень логирования для файла
            max_bytes: Максимальный размер файла в байтах
            backup_count: Количество резервных копий
            use_json: Использовать JSON формат для файлов
        """
        self.log_dir = Path(log_dir)
        self.app_name = app_name
        self.console_level = self._str_to_level(console_level)
        self.file_level = self._str_to_level(file_level)
        self.max_bytes = max_bytes
        self.backup_count = backup_count
        self.use_json = use_json
        
        # Хранилище созданных логгеров
        self._loggers = {}
        
        # Флаг настройки
        self._configured = False
    
    @staticmethod
    def _str_to_level(level: str) -> int:
        """Преобразует строку уровня в числовое значение."""
        levels = {
            'DEBUG': logging.DEBUG,
            'INFO': logging.INFO,
            'WARNING': logging.WARNING,
            'ERROR': logging.ERROR,
            'CRITICAL': logging.CRITICAL
        }
        return levels.get(level.upper(), logging.INFO)
    
    def _create_log_dir(self) -> None:
        """Создаёт папку для логов, если её нет."""
        self.log_dir.mkdir(parents=True, exist_ok=True)
    
    def _create_console_handler(self) -> logging.Handler:
        """Создаёт обработчик для вывода в консоль."""
        console_handler = logging.StreamHandler(sys.stdout)
        console_handler.setLevel(self.console_level)
        
        # Формат для консоли (цветной, краткий)
        console_format = '%(asctime)s - %(levelname)s - %(message)s'
        console_datefmt = '%H:%M:%S'
        
        console_handler.setFormatter(
            ColoredFormatter(console_format, console_datefmt)
        )
        
        return console_handler
    
    def _create_file_handler(self) -> logging.Handler:
        """Создаёт обработчик для вывода в файл с ротацией."""
        log_file = self.log_dir / f"{self.app_name}.log"
        
        file_handler = RotatingFileHandler(
            filename=log_file,
            maxBytes=self.max_bytes,
            backupCount=self.backup_count,
            encoding='utf-8'
        )
        file_handler.setLevel(self.file_level)
        
        # Формат для файла (подробный)
        if self.use_json:
            file_format = JsonFormatter()
        else:
            file_format = logging.Formatter(
                '%(asctime)s - %(name)s - %(levelname)s - %(module)s:%(lineno)d - %(funcName)s - %(message)s',
                datefmt='%Y-%m-%d %H:%M:%S'
            )
        
        file_handler.setFormatter(file_format)
        
        return file_handler
    
    def configure(self) -> None:
        """
        Настраивает корневой логгер.
        Вызывается один раз при запуске приложения.
        """
        if self._configured:
            return
        
        self._create_log_dir()
        
        # Очищаем существующие обработчики корневого логгера
        root_logger = logging.getLogger()
        root_logger.handlers.clear()
        
        # Устанавливаем минимальный уровень (самый низкий из всех)
        min_level = min(self.console_level, self.file_level)
        root_logger.setLevel(min_level)
        
        # Добавляем обработчики
        root_logger.addHandler(self._create_console_handler())
        root_logger.addHandler(self._create_file_handler())
        
        self._configured = True
        
        # Логируем успешную настройку
        logger = self.get_logger(__name__)
        logger.info(f"Система логирования настроена")
        logger.info(f"  Консоль: {logging.getLevelName(self.console_level)}")
        logger.info(f"  Файл: {logging.getLevelName(self.file_level)}")
        logger.info(f"  Папка логов: {self.log_dir.absolute()}")
    
    def get_logger(self, name: str) -> logging.Logger:
        """
        Возвращает настроенный логгер с указанным именем.
        
        Args:
            name: Имя логгера (обычно __name__)
        
        Returns:
            logging.Logger: Настроенный логгер
        """
        if not self._configured:
            self.configure()
        
        if name not in self._loggers:
            self._loggers[name] = logging.getLogger(name)
        
        return self._loggers[name]
    
    def set_level(self, level: str) -> None:
        """
        Динамически изменяет уровень логирования.
        
        Args:
            level: Новый уровень (DEBUG, INFO, WARNING, ERROR, CRITICAL)
        """
        new_level = self._str_to_level(level)
        
        # Изменяем уровень корневого логгера
        root_logger = logging.getLogger()
        root_logger.setLevel(new_level)
        
        # Изменяем уровень обработчиков
        for handler in root_logger.handlers:
            handler.setLevel(new_level)
        
        logger = self.get_logger(__name__)
        logger.info(f"Уровень логирования изменён на {level}")


# ============================================================
# ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ
# ============================================================

def get_level_from_env(default: str = "INFO") -> str:
    """Получает уровень логирования из переменной окружения LOG_LEVEL."""
    return os.getenv('LOG_LEVEL', default).upper()


def get_log_dir_from_env(default: str = "logs") -> str:
    """Получает папку для логов из переменной окружения LOG_DIR."""
    return os.getenv('LOG_DIR', default)


# ============================================================
# ФАБРИЧНАЯ ФУНКЦИЯ
# ============================================================

_default_setup = None


def setup_default_logging() -> LoggingSetup:
    """
    Создаёт и настраивает экземпляр LoggingSetup с параметрами по умолчанию.
    Использует переменные окружения для конфигурации.
    """
    global _default_setup
    
    if _default_setup is None:
        _default_setup = LoggingSetup(
            log_dir=get_log_dir_from_env(),
            console_level=get_level_from_env("INFO"),
            file_level=get_level_from_env("DEBUG"),
            use_json=os.getenv('LOG_JSON', 'false').lower() == 'true'
        )
        _default_setup.configure()
    
    return _default_setup


def get_logger(name: str) -> logging.Logger:
    """
    Быстрый способ получить настроенный логгер.
    
    Пример:
        from logger_setup import get_logger
        logger = get_logger(__name__)
        logger.info("Hello")
    """
    setup = setup_default_logging()
    return setup.get_logger(name)


# ============================================================
# ДЕМОНСТРАЦИЯ
# ============================================================

def demo():
    """Демонстрация работы системы логирования."""
    print("\n" + "=" * 70)
    print("ДЕМОНСТРАЦИЯ СИСТЕМЫ ЛОГИРОВАНИЯ")
    print("=" * 70)
    
    # Настройка через переменные окружения (можно переопределить)
    # os.environ['LOG_LEVEL'] = 'DEBUG'
    # os.environ['LOG_DIR'] = 'my_logs'
    
    # Получаем настроенный логгер
    logger = get_logger("demo")
    
    print("\n📋 Логирование с разными уровнями:\n")
    
    logger.debug("Отладочное сообщение (только в файл)")
    logger.info("Информационное сообщение")
    logger.warning("Предупреждение")
    logger.error("Ошибка")
    
    try:
        x = 1 / 0
    except ZeroDivisionError as e:
        logger.exception("Поймано исключение")
    
    logger.critical("Критическая ошибка")
    
    print("\n" + "=" * 70)
    print("Логи записаны в папку 'logs'")
    print("=" * 70)


if __name__ == "__main__":
    demo()
Ожидаемый вывод в консоли
text
======================================================================
ДЕМОНСТРАЦИЯ СИСТЕМЫ ЛОГИРОВАНИЯ
======================================================================

📋 Логирование с разными уровнями:

14:30:25 - INFO - Система логирования настроена
14:30:25 - INFO -   Консоль: INFO
14:30:25 - INFO -   Файл: DEBUG
14:30:25 - INFO -   Папка логов: /home/user/project/logs
14:30:25 - INFO - Информационное сообщение
14:30:25 - WARNING - Предупреждение
14:30:25 - ERROR - Ошибка
14:30:25 - ERROR - Поймано исключение
ZeroDivisionError: division by zero
14:30:25 - CRITICAL - Критическая ошибка

======================================================================
Логи записаны в папку 'logs'
======================================================================
Содержимое файла logs/app.log
text
2024-01-15 14:30:25 - logger_setup - INFO - logger_setup:123 - configure - Система логирования настроена
2024-01-15 14:30:25 - logger_setup - INFO - logger_setup:124 - configure -   Консоль: INFO
2024-01-15 14:30:25 - logger_setup - INFO - logger_setup:125 - configure -   Файл: DEBUG
2024-01-15 14:30:25 - logger_setup - INFO - logger_setup:126 - configure -   Папка логов: /home/user/project/logs
2024-01-15 14:30:25 - demo - DEBUG - <module>:203 - demo - Отладочное сообщение (только в файл)
2024-01-15 14:30:25 - demo - INFO - <module>:204 - demo - Информационное сообщение
2024-01-15 14:30:25 - demo - WARNING - <module>:205 - demo - Предупреждение
2024-01-15 14:30:25 - demo - ERROR - <module>:206 - demo - Ошибка
2024-01-15 14:30:25 - demo - ERROR - <module>:210 - demo - Поймано исключение
Traceback (most recent call last):
  File "logger_setup.py", line 208, in demo
    x = 1 / 0
ZeroDivisionError: division by zero
2024-01-15 14:30:25 - demo - CRITICAL - <module>:213 - demo - Критическая ошибка
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (консоль + файл, базовое форматирование)

Варианты 9-17: средний (ротация, цветной вывод, несколько файлов)

Варианты 18-25: сложный (JSON, база данных, распределённое логирование)

Варианты 1-8 (Базовый уровень)
№	Назначение	Особенности
1	Консольный калькулятор	Логировать все вычисления, ошибки ввода
2	Парсер CSV	Логировать процесс чтения и обработки строк
3	Валидатор email	Логировать результаты проверки
4	Конвертер валют	Логировать запросы к API
5	Генератор паролей	Логировать сгенерированные пароли (только в файл!)
6	Сканер портов	Логировать результаты сканирования
7	Копировщик файлов	Логировать все операции копирования
8	Таймер выполнения	Логировать время начала и окончания операций
Пример варианта 1:

python
# Требования:
# 1. Логировать все действия в консоль (INFO)
# 2. Логировать ошибки в файл (ERROR)
# 3. DEBUG логировать в отдельный файл

def setup_calculator_logging():
    # TODO: Реализовать настройку
    pass
Варианты 9-17 (Средний уровень)
№	Назначение	Особенности
9	Веб-сервер	Логировать запросы, ответы, ошибки
10	Telegram-бот	Логировать команды пользователей
11	Парсер сайтов	Логировать каждый запрос и ответ
12	Мониторинг серверов	Логировать проверки и алерты
13	Бэкап-система	Логировать успешные и неудачные бэкапы
14	Система авторизации	Логировать попытки входа (без паролей)
15	Кеширующий прокси	Логировать попадания/промахи кеша
16	Обработчик очередей	Логировать задачи и их выполнение
17	API клиент	Логировать запросы и ответы
Пример варианта 9:

python
# Требования:
# 1. Разные лог-файлы для:
#    - access.log (все запросы)
#    - error.log (только ошибки)
#    - debug.log (детальная информация)
# 2. Ротация логов раз в день

def setup_web_server_logging():
    # TODO: Реализовать настройку трёх обработчиков
    pass
Варианты 18-25 (Сложный уровень)
№	Назначение	Особенности
18	Микросервис	Логирование с correlation ID
19	ETL-пайплайн	Логирование каждого этапа обработки
20	Система с HIPAA	Логирование с маскированием чувствительных данных
21	Высоконагруженная система	Асинхронное логирование, буферизация
22	Распределённая система	Логирование с передачей в центральный сервер
23	ML-пайплайн	Логирование метрик и артефактов
24	IoT-платформа	Логирование миллионов устройств
25	Блокчейн-приложение	Логирование транзакций с верификацией
Пример варианта 18:

python
# Требования:
# 1. Каждый запрос имеет уникальный correlation_id
# 2. Все логи микросервиса содержат этот ID
# 3. Поддержка distributed tracing

class CorrelationIdFilter(logging.Filter):
    # TODO: Реализовать фильтр для добавления correlation_id
    pass
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Логирование не настроено или работает с ошибками
3 (удовлетворительно)	Настроен вывод в консоль и файл, есть базовое форматирование
4 (хорошо)	Настроены разные уровни, ротация, цветной вывод
5 (отлично)	Полная настройка, поддержка переменных окружения, JSON формат
5. Шпаргалка
5.1. Быстрая настройка
python
import logging

# Простейшая настройка
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.StreamHandler(),           # консоль
        logging.FileHandler('app.log')     # файл
    ]
)
5.2. Настройка с ротацией
python
from logging.handlers import RotatingFileHandler

handler = RotatingFileHandler(
    'app.log',
    maxBytes=1_048_576,  # 1 MB
    backupCount=5
)
5.3. Цветной вывод
python
class ColoredFormatter(logging.Formatter):
    COLORS = {
        'DEBUG': '\033[36m',
        'INFO': '\033[32m',
        'WARNING': '\033[33m',
        'ERROR': '\033[31m',
        'CRITICAL': '\033[35m',
    }
    
    def format(self, record):
        color = self.COLORS.get(record.levelname, '')
        record.levelname = f"{color}{record.levelname}\033[0m"
        return super().format(record)
5.4. Переменные окружения
python
import os

LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO').upper()
LOG_DIR = os.getenv('LOG_DIR', 'logs')
LOG_JSON = os.getenv('LOG_JSON', 'false').lower() == 'true'
5.5. Получение логгера по имени
python
def get_logger(name: str) -> logging.Logger:
    return logging.getLogger(name)

# В каждом модуле
logger = get_logger(__name__)
Карточка студента
text
ПЗ 2.16. НАСТРОЙКА СИСТЕМЫ ЛОГИРОВАНИЯ: ФАЙЛЫ, КОНСОЛЬ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== НАСТРОЙКИ ===

Консоль:
  Уровень: _____________
  Формат: ______________
  Цветной вывод: □ Да □ Нет

Файл:
  Путь: ________________
  Уровень: _____________
  Формат: ______________
  Ротация: □ Да □ Нет
  Размер: ______________
  Резервных копий: ______

=== ПРОВЕРКА ===

□ Логи выводятся в консоль
□ Логи выводятся в файл
□ Ротация работает корректно
□ Разные уровни логируются правильно
□ Исключения логируются с traceback

=== ОТЧЁТ ===

Файл конфигурации: _____________
Скриншот консоли: _____________
Содержимое лог-файла: _____________

Дата выполнения: _____________
