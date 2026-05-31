# ПЗ 2.42. Настройка иерархии логгеров для разных модулей

**Тема:** Продвинутое логирование, иерархия логгеров, фильтры, конфигурация

**Цель работы:**  
Научиться настраивать иерархию логгеров для разных модулей приложения, использовать фильтры для гибкой маршрутизации сообщений, настраивать разные уровни логирования для разных компонентов.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+

> Главная мысль: **Один логгер на всё приложение — это хаос. Иерархия логгеров позволяет контролировать поток сообщений от каждого модуля отдельно.**

---

## Содержание

1. [Теоретическая справка](#1-теоретическая-справка)
2. [Нулевой вариант (эталонный)](#2-нулевой-вариант-эталонный)
3. [25 вариантов практической работы](#3-25-вариантов-практической-работы)
4. [Критерии оценки](#4-критерии-оценки)
5. [Шпаргалка](#5-шпаргалка)

---

## 1. Теоретическая справка

### 1.1. Принцип иерархии логгеров

Логгеры в Python организованы в иерархическую структуру, где имена разделяются точками.
┌─────────────────────────────────────────────────────────────────────────────┐
│ ИЕРАРХИЯ ЛОГГЕРОВ В ПРИЛОЖЕНИИ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ┌─────────────────┐ │
│ │ root │ │
│ │ (корневой) │ │
│ └────────┬────────┘ │
│ │ │
│ ┌──────────────────┼──────────────────┐ │
│ │ │ │ │
│ ▼ ▼ ▼ │
│ ┌───────────┐ ┌───────────┐ ┌───────────┐ │
│ │ app │ │ database │ │ api │ │
│ │ (WARNING) │ │ (INFO) │ │ (DEBUG) │ │
│ └─────┬─────┘ └───────────┘ └─────┬─────┘ │
│ │ │ │
│ ┌──────────┼──────────┐ │ │
│ │ │ │ │ │
│ ▼ ▼ ▼ ▼ │
│ ┌────────┐ ┌────────┐ ┌────────┐ ┌───────────┐ │
│ │ models │ │ views │ │ utils │ │ client │ │
│ │(DEBUG) │ │(INFO) │ │(ERROR) │ │ (WARNING) │ │
│ └────────┘ └────────┘ └────────┘ └───────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Правила наследования

| Правило | Описание |
|---------|----------|
| **Уровень** | Если у логгера не установлен уровень, он наследуется от родителя |
| **Обработчики** | Сообщения передаются всем обработчикам логгера и его предков |
| **Propagate** | Если `propagate=False`, сообщение не передаётся родителю |
| **Каскадирование** | Сообщение может быть обработано несколькими обработчиками |

---

## 2. Нулевой вариант (эталонный)

**Назначение:** преподаватель выполняет этот вариант на занятии, показывая настройку иерархии логгеров.

### Техническое задание (нулевой вариант)

> Разработайте систему логирования для веб-приложения с модулями:
> - `app` — основное приложение (WARNING)
> - `app.auth` — модуль авторизации (INFO)
> - `app.database` — модуль базы данных (DEBUG)
> - `app.api` — API-клиент (ERROR)
> - `app.utils` — утилиты (INFO)
>
> Настройте:
> - Разные уровни логирования для каждого модуля
> - Разные обработчики для разных типов сообщений
> - Фильтры для разделения логов по модулям

### Эталонная реализация

#### Структура проекта
my_app/
├── main.py
├── logging_config.py
├── modules/
│ ├── init.py
│ ├── auth.py
│ ├── database.py
│ ├── api.py
│ └── utils.py
└── logs/
├── app.log
├── auth.log
├── database.log
├── api.log
└── errors.log

text

#### Файл `logging_config.py`

```python
#!/usr/bin/env python3
"""
logging_config.py — Настройка иерархии логгеров для приложения.
"""

import logging
import logging.config
from pathlib import Path
from datetime import datetime
import sys


# ============================================================
# КАСТОМНЫЕ ФИЛЬТРЫ
# ============================================================

class ModuleFilter(logging.Filter):
    """Фильтр для логов по имени модуля."""
    
    def __init__(self, module_prefix: str):
        super().__init__()
        self.module_prefix = module_prefix
    
    def filter(self, record):
        return record.name.startswith(self.module_prefix)


class LevelRangeFilter(logging.Filter):
    """Фильтр для диапазона уровней."""
    
    def __init__(self, min_level: int, max_level: int):
        super().__init__()
        self.min_level = min_level
        self.max_level = max_level
    
    def filter(self, record):
        return self.min_level <= record.levelno <= self.max_level


class ContextFilter(logging.Filter):
    """Фильтр, добавляющий контекстную информацию."""
    
    def filter(self, record):
        record.user_id = getattr(record, 'user_id', 'anonymous')
        record.session_id = getattr(record, 'session_id', '-')
        record.request_id = getattr(record, 'request_id', '-')
        return True


# ============================================================
# КАСТОМНЫЕ ФОРМАТТЕРЫ
# ============================================================

class ColoredFormatter(logging.Formatter):
    """Цветной форматтер для консоли."""
    
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


class DetailedFormatter(logging.Formatter):
    """Подробный форматтер с контекстом."""
    
    def format(self, record):
        # Добавляем время выполнения если есть
        if hasattr(record, 'execution_time'):
            record.execution_time = f"{record.execution_time:.3f}s"
        else:
            record.execution_time = '-'
        
        return super().format(record)


# ============================================================
# НАСТРОЙКА ЛОГГЕРОВ
# ============================================================

class AppLoggerConfig:
    """Конфигуратор иерархии логгеров для приложения."""
    
    def __init__(self, log_dir: str = "logs"):
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(exist_ok=True)
        
        # Создаём поддиректории
        (self.log_dir / "app").mkdir(exist_ok=True)
        (self.log_dir / "auth").mkdir(exist_ok=True)
        (self.log_dir / "database").mkdir(exist_ok=True)
        (self.log_dir / "api").mkdir(exist_ok=True)
        (self.log_dir / "errors").mkdir(exist_ok=True)
    
    def setup(self):
        """Настройка всей иерархии логгеров."""
        
        # ============================================================
        # КОРНЕВОЙ ЛОГГЕР
        # ============================================================
        root_logger = logging.getLogger()
        root_logger.setLevel(logging.DEBUG)
        
        # Удаляем существующие обработчики
        for handler in root_logger.handlers[:]:
            root_logger.removeHandler(handler)
        
        # ============================================================
        # ОБЩИЙ ОБРАБОТЧИК ДЛЯ ОШИБОК
        # ============================================================
        error_handler = logging.FileHandler(
            self.log_dir / "errors" / "all_errors.log",
            encoding='utf-8'
        )
        error_handler.setLevel(logging.ERROR)
        error_handler.setFormatter(DetailedFormatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s\n'
        ))
        root_logger.addHandler(error_handler)
        
        # ============================================================
        # КОНСОЛЬНЫЙ ОБРАБОТЧИК (только INFO и выше)
        # ============================================================
        console_handler = logging.StreamHandler(sys.stdout)
        console_handler.setLevel(logging.INFO)
        console_handler.setFormatter(ColoredFormatter(
            '%(asctime)s - %(levelname)s - %(message)s',
            datefmt='%H:%M:%S'
        ))
        root_logger.addHandler(console_handler)
        
        # ============================================================
        # НАСТРОЙКА ЛОГГЕРА APP
        # ============================================================
        app_logger = logging.getLogger('app')
        app_logger.setLevel(logging.WARNING)
        app_logger.propagate = True  # Передаём в root
        
        # Файл для app (только WARNING и выше)
        app_handler = logging.FileHandler(
            self.log_dir / "app" / "app.log",
            encoding='utf-8'
        )
        app_handler.setLevel(logging.WARNING)
        app_handler.setFormatter(logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        ))
        app_logger.addHandler(app_handler)
        
        # ============================================================
        # НАСТРОЙКА ЛОГГЕРА AUTH
        # ============================================================
        auth_logger = logging.getLogger('app.auth')
        auth_logger.setLevel(logging.INFO)
        auth_logger.propagate = True
        
        # Отдельный файл для auth
        auth_handler = logging.FileHandler(
            self.log_dir / "auth" / "auth.log",
            encoding='utf-8'
        )
        auth_handler.setLevel(logging.INFO)
        auth_handler.setFormatter(logging.Formatter(
            '%(asctime)s - %(levelname)s - %(user_id)s - %(message)s'
        ))
        auth_logger.addHandler(auth_handler)
        
        # Добавляем контекстный фильтр
        auth_logger.addFilter(ContextFilter())
        
        # ============================================================
        # НАСТРОЙКА ЛОГГЕРА DATABASE
        # ============================================================
        db_logger = logging.getLogger('app.database')
        db_logger.setLevel(logging.DEBUG)
        db_logger.propagate = True
        
        # Отдельный файл для database (все уровни)
        db_handler = logging.FileHandler(
            self.log_dir / "database" / "database.log",
            encoding='utf-8'
        )
        db_handler.setLevel(logging.DEBUG)
        db_handler.setFormatter(DetailedFormatter(
            '%(asctime)s - %(levelname)s - %(module)s:%(lineno)d - %(message)s'
        ))
        db_logger.addHandler(db_handler)
        
        # ============================================================
        # НАСТРОЙКА ЛОГГЕРА API
        # ============================================================
        api_logger = logging.getLogger('app.api')
        api_logger.setLevel(logging.ERROR)
        api_logger.propagate = True
        
        # Отдельный файл для api (только ERROR)
        api_handler = logging.FileHandler(
            self.log_dir / "api" / "api.log",
            encoding='utf-8'
        )
        api_handler.setLevel(logging.ERROR)
        api_handler.setFormatter(logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s\n'
        ))
        api_logger.addHandler(api_handler)
        
        # ============================================================
        # НАСТРОЙКА ЛОГГЕРА UTILS
        # ============================================================
        utils_logger = logging.getLogger('app.utils')
        utils_logger.setLevel(logging.INFO)
        utils_logger.propagate = False  # Не передаём в root
        
        # Отдельный файл для utils
        utils_handler = logging.FileHandler(
            self.log_dir / "app" / "utils.log",
            encoding='utf-8'
        )
        utils_handler.setLevel(logging.INFO)
        utils_handler.setFormatter(logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s'
        ))
        utils_logger.addHandler(utils_handler)
        
        # ============================================================
        # СПЕЦИАЛЬНЫЙ ФИЛЬТР ДЛЯ SQLALCHEMY
        # ============================================================
        sql_logger = logging.getLogger('sqlalchemy.engine')
        sql_logger.setLevel(logging.WARNING)
        
        # Добавляем фильтр для sqlalchemy (только WARNING и ERROR)
        sql_handler = logging.FileHandler(
            self.log_dir / "database" / "sqlalchemy.log",
            encoding='utf-8'
        )
        sql_handler.setLevel(logging.WARNING)
        sql_logger.addHandler(sql_handler)
    
    def get_logger(self, name: str) -> logging.Logger:
        """Получение настроенного логгера."""
        return logging.getLogger(name)


# ============================================================
# МОДУЛИ ПРИЛОЖЕНИЯ
# ============================================================

# В реальном проекте эти классы находятся в разных файлах

class AuthModule:
    """Модуль авторизации."""
    
    def __init__(self, logger: logging.Logger):
        self.logger = logger
    
    def login(self, user_id: str, password: str) -> bool:
        self.logger.info(f"Попытка входа пользователя {user_id}")
        
        if len(password) < 6:
            self.logger.warning(f"Слабый пароль для {user_id}")
            return False
        
        # Имитация успешного входа
        self.logger.info(f"Успешный вход пользователя {user_id}")
        return True
    
    def logout(self, user_id: str):
        self.logger.info(f"Выход пользователя {user_id}")


class DatabaseModule:
    """Модуль базы данных."""
    
    def __init__(self, logger: logging.Logger):
        self.logger = logger
    
    def connect(self):
        self.logger.debug("Попытка подключения к БД")
        # Имитация подключения
        self.logger.info("Подключение к БД установлено")
    
    def query(self, sql: str):
        self.logger.debug(f"Выполнение запроса: {sql[:50]}...")
        # Имитация выполнения
        self.logger.debug("Запрос выполнен успешно")
    
    def disconnect(self):
        self.logger.info("Отключение от БД")


class APIModule:
    """API-клиент."""
    
    def __init__(self, logger: logging.Logger):
        self.logger = logger
    
    def call(self, endpoint: str):
        self.logger.debug(f"Вызов API: {endpoint}")
        
        # Имитация ошибки
        import random
        if random.random() < 0.3:
            self.logger.error(f"Ошибка API при вызове {endpoint}")
            return False
        
        self.logger.info(f"API вызов {endpoint} успешен")
        return True


class UtilsModule:
    """Утилиты."""
    
    def __init__(self, logger: logging.Logger):
        self.logger = logger
    
    def helper_function(self):
        self.logger.debug("Вызов вспомогательной функции")
        self.logger.info("Вспомогательная функция выполнена")


# ============================================================
# ГЛАВНАЯ ФУНКЦИЯ
# ============================================================

def main():
    """Демонстрация работы иерархии логгеров."""
    
    print("=" * 70)
    print("НАСТРОЙКА ИЕРАРХИИ ЛОГГЕРОВ ДЛЯ РАЗНЫХ МОДУЛЕЙ")
    print("=" * 70)
    
    # Настройка логирования
    config = AppLoggerConfig()
    config.setup()
    
    # Получение логгеров для разных модулей
    app_logger = config.get_logger('app')
    auth_logger = config.get_logger('app.auth')
    db_logger = config.get_logger('app.database')
    api_logger = config.get_logger('app.api')
    utils_logger = config.get_logger('app.utils')
    
    # Создание модулей
    auth = AuthModule(auth_logger)
    db = DatabaseModule(db_logger)
    api = APIModule(api_logger)
    utils = UtilsModule(utils_logger)
    
    # ============================================================
    # ДЕМОНСТРАЦИЯ
    # ============================================================
    
    print("\n📋 1. Логирование в модуле APP")
    print("-" * 50)
    app_logger.warning("Предупреждение из основного приложения")
    app_logger.error("Ошибка в основном приложении")
    
    print("\n🔐 2. Логирование в модуле AUTH")
    print("-" * 50)
    auth.login("user123", "weak")
    auth.login("user456", "strong_password")
    auth.logout("user123")
    
    print("\n💾 3. Логирование в модуле DATABASE")
    print("-" * 50)
    db.connect()
    db.query("SELECT * FROM users WHERE id = 1")
    db.disconnect()
    
    print("\n🌐 4. Логирование в модуле API")
    print("-" * 50)
    for i in range(5):
        api.call(f"/api/users/{i}")
    
    print("\n🛠️ 5. Логирование в модуле UTILS")
    print("-" * 50)
    utils.helper_function()
    
    # ============================================================
    # ДЕМОНСТРАЦИЯ НАСЛЕДОВАНИЯ УРОВНЕЙ
    # ============================================================
    
    print("\n" + "=" * 70)
    print("ДЕМОНСТРАЦИЯ НАСЛЕДОВАНИЯ УРОВНЕЙ")
    print("=" * 70)
    
    print("\n📊 Уровни логгеров:")
    print(f"   root: {logging.getLogger().level}")
    print(f"   app: {logging.getLogger('app').level}")
    print(f"   app.auth: {logging.getLogger('app.auth').level}")
    print(f"   app.database: {logging.getLogger('app.database').level}")
    
    print("\n🔄 Propagate настройки:")
    print(f"   app.utils.propagate = {logging.getLogger('app.utils').propagate}")
    print(f"   app.auth.propagate = {logging.getLogger('app.auth').propagate}")
    
    # Демонстрация каскадирования
    print("\n📨 Тест каскадирования сообщений:")
    test_logger = logging.getLogger('app.test')
    test_logger.info("Это сообщение пойдёт в корневой логгер, т.к. у test нет обработчиков")
    
    print("\n" + "=" * 70)
    print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
    print("=" * 70)
    print("\n📁 Проверьте файлы в директории logs/ для просмотра результатов")
    print("   - logs/app/app.log - предупреждения приложения")
    print("   - logs/auth/auth.log - логи авторизации с user_id")
    print("   - logs/database/database.log - подробные логи БД")
    print("   - logs/api/api.log - ошибки API")
    print("   - logs/errors/all_errors.log - все ошибки приложения")


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
======================================================================
НАСТРОЙКА ИЕРАРХИИ ЛОГГЕРОВ ДЛЯ РАЗНЫХ МОДУЛЕЙ
======================================================================

📋 1. Логирование в модуле APP
--------------------------------------------------
14:30:25 - WARNING - Предупреждение из основного приложения
14:30:25 - ERROR - Ошибка в основном приложении

🔐 2. Логирование в модуле AUTH
--------------------------------------------------
14:30:25 - INFO - Попытка входа пользователя user123
14:30:25 - WARNING - Слабый пароль для user123
14:30:25 - INFO - Попытка входа пользователя user456
14:30:25 - INFO - Успешный вход пользователя user456
14:30:25 - INFO - Выход пользователя user123

💾 3. Логирование в модуле DATABASE
--------------------------------------------------
14:30:25 - INFO - Подключение к БД установлено
14:30:25 - DEBUG - Выполнение запроса: SELECT * FROM users WHERE id = 1...
14:30:25 - DEBUG - Запрос выполнен успешно
14:30:25 - INFO - Отключение от БД

🌐 4. Логирование в модуле API
--------------------------------------------------
14:30:25 - INFO - API вызов /api/users/0 успешен
14:30:25 - INFO - API вызов /api/users/1 успешен
14:30:25 - ERROR - Ошибка API при вызове /api/users/2
14:30:25 - INFO - API вызов /api/users/3 успешен
14:30:25 - INFO - API вызов /api/users/4 успешен

🛠️ 5. Логирование в модуле UTILS
--------------------------------------------------
14:30:25 - INFO - Вспомогательная функция выполнена

======================================================================
ДЕМОНСТРАЦИЯ НАСЛЕДОВАНИЯ УРОВНЕЙ
======================================================================

📊 Уровни логгеров:
   root: 10
   app: 30
   app.auth: 20
   app.database: 10

🔄 Propagate настройки:
   app.utils.propagate = False
   app.auth.propagate = True

📨 Тест каскадирования сообщений:
14:30:25 - INFO - Это сообщение пойдёт в корневой логгер, т.к. у test нет обработчиков

======================================================================
ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА
======================================================================

📁 Проверьте файлы в директории logs/ для просмотра результатов
   - logs/app/app.log - предупреждения приложения
   - logs/auth/auth.log - логи авторизации с user_id
   - logs/database/database.log - подробные логи БД
   - logs/api/api.log - ошибки API
   - logs/errors/all_errors.log - все ошибки приложения
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (простая иерархия, 3-4 модуля)

Варианты 9-17: средний (разные уровни, фильтры, propagate)

Варианты 18-25: сложный (много модулей, кастомные фильтры)

Варианты 1-8 (Базовый уровень)
№	Модули	Уровни	Особенности
1	app, db, utils	INFO, DEBUG, WARNING	Базовая иерархия
2	main, auth, data	WARNING, INFO, DEBUG	Разные обработчики
3	server, client, cache	ERROR, INFO, DEBUG	Консоль + файл
4	web, api, storage	INFO, DEBUG, WARNING	Ротация по размеру
5	processor, queue, worker	DEBUG, INFO, ERROR	Обработчики по модулям
6	parser, analyzer, exporter	INFO, WARNING, DEBUG	Разные форматы
7	fetcher, validator, saver	ERROR, DEBUG, INFO	Фильтры по уровню
8	scheduler, executor, logger	INFO, DEBUG, WARNING	Многоуровневая
Варианты 9-17 (Средний уровень)
№	Модули	Дополнительные требования
9	app, auth, db, api, utils	Разные файлы для каждого
10	web, auth, db, cache, queue	Фильтры по модулям
11	server, handler, middleware, service	Propagate настройки
12	controller, service, repository, model	Иерархия 3+ уровней
13	cli, config, plugin, hook, event	Динамическое изменение уровней
14	producer, consumer, broker, worker	Контекстные фильтры
15	extractor, transformer, loader, validator	Кастомные форматеры
16	collector, aggregator, reporter, notifier	Ротация по времени
17	monitor, alert, metric, dashboard	Разные форматы вывода
Варианты 18-25 (Сложный уровень)
№	Тема	Особенности
18	Микросервисная архитектура	Много сервисов, централизованный сбор
19	Веб-приложение с плагинами	Динамическая загрузка модулей
20	Система с фоновыми задачами	Асинхронное логирование
21	Распределённая система	Correlation ID, трассировка
22	ETL-пайплайн	Много этапов, агрегация логов
23	API-шлюз	Логирование запросов, метрик
24	Система мониторинга	Сбор логов, алерты
25	Big Data обработка	Много потоков, сжатие
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Иерархия не настроена
3 (удовлетворительно)	Настроены 2-3 логгера с разными уровнями
4 (хорошо)	+ разные обработчики, propagate
5 (отлично)	+ фильтры, контекст, отдельные файлы
5. Шпаргалка
python
# === ИЕРАРХИЯ ЛОГГЕРОВ ===
root = logging.getLogger()
app = logging.getLogger('app')
db = logging.getLogger('app.database')
api = logging.getLogger('app.api')

# === УРОВНИ ===
app.setLevel(logging.WARNING)
db.setLevel(logging.DEBUG)
api.setLevel(logging.ERROR)

# === PROPAGATE ===
app.propagate = True   # передавать в root
utils.propagate = False # не передавать

# === ФИЛЬТРЫ ===
class ModuleFilter(logging.Filter):
    def __init__(self, prefix):
        self.prefix = prefix
    def filter(self, record):
        return record.name.startswith(self.prefix)

# === ПОЛУЧЕНИЕ ЛОГГЕРА ===
logger = logging.getLogger(__name__)  # в модуле
Карточка студента
text
ПЗ 2.42. НАСТРОЙКА ИЕРАРХИИ ЛОГГЕРОВ ДЛЯ РАЗНЫХ МОДУЛЕЙ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== МОДУЛИ ===

1. _____________ (уровень: _____)
2. _____________ (уровень: _____)
3. _____________ (уровень: _____)
4. _____________ (уровень: _____)
5. _____________ (уровень: _____)

=== КОМПОНЕНТЫ ===

□ Корневой логгер
□ Логгеры для каждого модуля
□ Разные уровни логирования
□ Обработчики (файлы, консоль)
□ Фильтры по модулям
□ Propagate настройки
□ Контекстная информация

=== ОТЧЁТ ===

Количество логгеров: _____
Количество файлов логов: _____
Propagate включён: □ Да □ Нет

Дата выполнения: _____________
