# Тема 2.36. Ротация логов. Настройка для больших проектов

**Цель лекции:**  
Изучить механизмы ротации логов в Python, настроить автоматическое управление файлами логов для больших проектов, освоить различные стратегии ротации и архивации.

> Главная мысль: **Логи имеют свойство заполнять диск. Ротация логов — это автоматический механизм, который заботится о том, чтобы логи не «съели» всё свободное место.**

---

## Содержание

1. [Введение в ротацию логов](#1-введение-в-ротацию-логов)
2. [RotatingFileHandler — ротация по размеру](#2-rotatingfilehandler-ротация-по-размеру)
3. [TimedRotatingFileHandler — ротация по времени](#3-timedrotatingfilehandler-ротация-по-времени)
4. [Продвинутые стратегии ротации](#4-продвинутые-стратегии-ротации)
5. [Настройка для больших проектов](#5-настройка-для-больших-проектов)
6. [Практические примеры](#6-практические-примеры)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Введение в ротацию логов

### 1.1. Что такое ротация логов

**Ротация логов** — это процесс управления файлами логов, включающий создание новых файлов и архивацию старых.
┌─────────────────────────────────────────────────────────────────────────────┐
│ ПРОЦЕСС РОТАЦИИ ЛОГОВ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ app.log (текущий) │
│ │ │
│ ▼ (достигнут лимит) │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ app.log → app.log.1 → app.log.2 → app.log.3 │ │
│ │ (новый) (старый 1) (старый 2) (старый 3) │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
│ или │
│ │
│ app.log (текущий) │
│ │ │
│ ▼ (наступило время) │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ app.log → app.2024-01-15.log │ │
│ │ (новый) (старый по дате) │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Почему нужна ротация

| Проблема | Решение |
|----------|---------|
| **Заполнение диска** | Ограничение размера файлов |
| **Медленный поиск** | Разделение по времени |
| **Аналитика** | Удобная архивация по датам |
| **Производительность** | Не нужно работать с гигантскими файлами |
| **Соответствие политикам** | Хранение логов заданный срок |

### 1.3. Стратегии ротации

| Стратегия | Описание | Когда использовать |
|------------|----------|---------------------|
| **По размеру** | Новый файл при достижении лимита | Высокая нагрузка, много логов |
| **По времени** | Новый файл каждый день/час | Логирование по датам для аналитики |
| **Комбинированная** | И по размеру, и по времени | Крупные проекты |
| **По количеству** | Ограничение количества файлов | Экономия места |

---

## 2. RotatingFileHandler — ротация по размеру

### 2.1. Базовое использование

```python
import logging
from logging.handlers import RotatingFileHandler
import time


# Создание обработчика с ротацией по размеру
handler = RotatingFileHandler(
    'app.log',          # имя файла
    maxBytes=1024,      # максимальный размер в байтах (1 KB для демонстрации)
    backupCount=3,      # количество резервных копий
    encoding='utf-8'
)

# Настройка форматтера
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

# Добавление к логгеру
logger = logging.getLogger(__name__)
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

# Генерация сообщений
for i in range(100):
    logger.info(f"Сообщение номер {i}")
    time.sleep(0.01)

print("Готово. Проверьте файлы: app.log, app.log.1, app.log.2, app.log.3")
2.2. Настройка для реальных проектов
python
import logging
from logging.handlers import RotatingFileHandler
from pathlib import Path


class RotatingFileLogger:
    """Класс для настройки ротации логов по размеру."""
    
    def __init__(self, log_dir: str = "logs", max_mb: int = 10, backup_count: int = 5):
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(exist_ok=True)
        self.max_bytes = max_mb * 1024 * 1024  # MB в байты
        self.backup_count = backup_count
    
    def setup_logger(self, name: str = "app") -> logging.Logger:
        """Настройка логгера с ротацией."""
        logger = logging.getLogger(name)
        
        # Удаляем существующие обработчики, чтобы не дублировать
        if logger.handlers:
            logger.handlers.clear()
        
        # Создаём обработчик
        handler = RotatingFileHandler(
            filename=self.log_dir / f"{name}.log",
            maxBytes=self.max_bytes,
            backupCount=self.backup_count,
            encoding='utf-8'
        )
        
        # Форматтер
        formatter = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(module)s:%(lineno)d - %(message)s',
            datefmt='%Y-%m-%d %H:%M:%S'
        )
        handler.setFormatter(formatter)
        
        logger.addHandler(handler)
        logger.setLevel(logging.DEBUG)
        
        # Добавляем также консольный вывод для разработки
        console = logging.StreamHandler()
        console.setLevel(logging.INFO)
        console.setFormatter(logging.Formatter('%(levelname)s: %(message)s'))
        logger.addHandler(console)
        
        return logger


# Использование
log_config = RotatingFileLogger(log_dir="logs", max_mb=5, backup_count=10)
logger = log_config.setup_logger("my_app")

logger.info("Приложение запущено")
logger.debug("Отладочное сообщение")
logger.error("Ошибка", exc_info=True)
3. TimedRotatingFileHandler — ротация по времени
3.1. Базовое использование
python
import logging
from logging.handlers import TimedRotatingFileHandler
import time


# Создание обработчика с ротацией по времени
handler = TimedRotatingFileHandler(
    'app.log',          # имя файла
    when='S',           # ротация каждую секунду (для демонстрации)
    interval=1,         # интервал = 1 секунда
    backupCount=5,      # хранить 5 файлов
    encoding='utf-8'
)

# Настройка форматтера
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

# Добавление к логгеру
logger = logging.getLogger(__name__)
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

# Генерация сообщений в течение 10 секунд
start = time.time()
while time.time() - start < 10:
    logger.info(f"Сообщение в {time.strftime('%H:%M:%S')}")
    time.sleep(0.5)

print("Готово. Проверьте файлы: app.log, app.log.2024-01-15_14-30-25 и т.д.")
3.2. Параметры ротации по времени
Параметр when	Описание	Интервал
'S'	Секунды	Каждые interval секунд
'M'	Минуты	Каждые interval минут
'H'	Часы	Каждые interval часов
'D'	Дни	Каждые interval дней
'W0' - 'W6'	Дни недели	В указанный день недели
'midnight'	Полночь	Каждый день в 00:00
3.3. Настройка для продакшена
python
import logging
from logging.handlers import TimedRotatingFileHandler
from pathlib import Path
from datetime import datetime


class TimedRotatingLogger:
    """Класс для настройки ротации логов по времени."""
    
    def __init__(self, log_dir: str = "logs", when: str = "midnight", backup_days: int = 30):
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(exist_ok=True)
        self.when = when
        self.backup_days = backup_days
    
    def setup_logger(self, name: str = "app") -> logging.Logger:
        """Настройка логгера с временной ротацией."""
        logger = logging.getLogger(name)
        
        if logger.handlers:
            logger.handlers.clear()
        
        # Основной обработчик (ежедневная ротация)
        handler = TimedRotatingFileHandler(
            filename=self.log_dir / f"{name}.log",
            when=self.when,
            interval=1,
            backupCount=self.backup_days,  # хранить 30 дней
            encoding='utf-8',
            utc=False
        )
        
        # Добавляем суффикс для старых файлов
        handler.suffix = "%Y-%m-%d"
        
        # Форматтер
        formatter = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        )
        handler.setFormatter(formatter)
        
        logger.addHandler(handler)
        logger.setLevel(logging.INFO)
        
        return logger


# Использование
log_config = TimedRotatingLogger(log_dir="logs", when="midnight", backup_days=30)
logger = log_config.setup_logger("my_app")

logger.info("Ежедневный лог-файл")
4. Продвинутые стратегии ротации
4.1. Комбинированная ротация (размер + время)
python
import logging
from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler


class CombinedRotatingLogger:
    """Комбинированная ротация: и по размеру, и по времени."""
    
    def __init__(self, log_dir: str = "logs", max_mb: int = 100, when: str = "midnight"):
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(exist_ok=True)
        self.max_bytes = max_mb * 1024 * 1024
        self.when = when
    
    def setup_logger(self, name: str = "app") -> logging.Logger:
        """Настройка логгера с комбинированной ротацией."""
        logger = logging.getLogger(name)
        
        if logger.handlers:
            logger.handlers.clear()
        
        # 1. Ротация по размеру (для текущего дня)
        size_handler = RotatingFileHandler(
            filename=self.log_dir / f"{name}.log",
            maxBytes=self.max_bytes,
            backupCount=5,
            encoding='utf-8'
        )
        
        # 2. Ротация по времени (для архивации по дням)
        time_handler = TimedRotatingFileHandler(
            filename=self.log_dir / f"{name}_daily.log",
            when=self.when,
            interval=1,
            backupCount=30,
            encoding='utf-8'
        )
        time_handler.suffix = "%Y-%m-%d"
        
        # Общий форматтер
        formatter = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        )
        size_handler.setFormatter(formatter)
        time_handler.setFormatter(formatter)
        
        # Устанавливаем разные уровни
        size_handler.setLevel(logging.DEBUG)   # в основной файл всё
        time_handler.setLevel(logging.INFO)    # в ежедневный только INFO+
        
        logger.addHandler(size_handler)
        logger.addHandler(time_handler)
        logger.setLevel(logging.DEBUG)
        
        return logger
4.2. Обработчик с сжатием старых логов
python
import logging
import gzip
import shutil
from logging.handlers import RotatingFileHandler
from pathlib import Path


class CompressedRotatingFileHandler(RotatingFileHandler):
    """Обработчик с автоматическим сжатием старых файлов."""
    
    def __init__(self, filename, maxBytes=0, backupCount=0, encoding=None, compress=True):
        super().__init__(filename, maxBytes, backupCount, encoding)
        self.compress = compress
    
    def doRollover(self):
        """Переопределяем метод ротации для сжатия."""
        super().doRollover()
        
        if self.compress:
            # Сжимаем самый старый файл
            for i in range(self.backupCount, 0, -1):
                src_name = f"{self.baseFilename}.{i}"
                dst_name = f"{src_name}.gz"
                
                if Path(src_name).exists() and not Path(dst_name).exists():
                    with open(src_name, 'rb') as f_in:
                        with gzip.open(dst_name, 'wb') as f_out:
                            shutil.copyfileobj(f_in, f_out)
                    Path(src_name).unlink()
                    break


# Использование
handler = CompressedRotatingFileHandler(
    'app.log',
    maxBytes=1024 * 1024,  # 1 MB
    backupCount=10,
    compress=True
)
4.3. Обработчик с ограничением общего размера
python
import logging
import os
from logging.handlers import RotatingFileHandler
from pathlib import Path


class TotalSizeLimitedHandler(RotatingFileHandler):
    """
    Обработчик, ограничивающий общий размер всех лог-файлов.
    """
    
    def __init__(self, filename, maxBytes=0, backupCount=0, totalSizeLimit=0, **kwargs):
        super().__init__(filename, maxBytes, backupCount, **kwargs)
        self.totalSizeLimit = totalSizeLimit
    
    def doRollover(self):
        super().doRollover()
        
        if self.totalSizeLimit > 0:
            self._enforce_total_size_limit()
    
    def _enforce_total_size_limit(self):
        """Удаляет старые файлы, если общий размер превышает лимит."""
        log_dir = Path(self.baseFilename).parent
        log_files = sorted(
            log_dir.glob(f"{Path(self.baseFilename).stem}*"),
            key=lambda f: f.stat().st_mtime,
            reverse=True
        )
        
        total_size = sum(f.stat().st_size for f in log_files)
        
        while total_size > self.totalSizeLimit and len(log_files) > 1:
            oldest = log_files.pop()
            total_size -= oldest.stat().st_size
            oldest.unlink()
            print(f"Удалён старый лог-файл: {oldest.name}")
5. Настройка для больших проектов
5.1. Конфигурация через словарь
python
import logging.config
import os
from pathlib import Path


def setup_production_logging():
    """Настройка логирования для production-проекта."""
    
    # Создаём директории
    log_dir = Path("logs")
    log_dir.mkdir(exist_ok=True)
    (log_dir / "app").mkdir(exist_ok=True)
    (log_dir / "access").mkdir(exist_ok=True)
    (log_dir / "error").mkdir(exist_ok=True)
    
    LOGGING_CONFIG = {
        'version': 1,
        'disable_existing_loggers': False,
        
        'formatters': {
            'detailed': {
                'format': '%(asctime)s - %(name)s - %(levelname)s - %(module)s:%(lineno)d - %(message)s',
                'datefmt': '%Y-%m-%d %H:%M:%S'
            },
            'access': {
                'format': '%(asctime)s - %(remote_addr)s - %(request_method)s %(request_path)s - %(status)s - %(duration).3fms',
                'datefmt': '%Y-%m-%d %H:%M:%S'
            },
            'simple': {
                'format': '%(asctime)s - %(levelname)s - %(message)s'
            }
        },
        
        'handlers': {
            # Application logs (ротация по размеру)
            'app_file': {
                'class': 'logging.handlers.RotatingFileHandler',
                'level': 'DEBUG',
                'formatter': 'detailed',
                'filename': 'logs/app/app.log',
                'maxBytes': 50 * 1024 * 1024,  # 50 MB
                'backupCount': 10,
                'encoding': 'utf-8'
            },
            
            # Access logs (ротация по времени)
            'access_file': {
                'class': 'logging.handlers.TimedRotatingFileHandler',
                'level': 'INFO',
                'formatter': 'access',
                'filename': 'logs/access/access.log',
                'when': 'midnight',
                'backupCount': 30,
                'encoding': 'utf-8'
            },
            
            # Error logs (только ошибки)
            'error_file': {
                'class': 'logging.handlers.TimedRotatingFileHandler',
                'level': 'ERROR',
                'formatter': 'detailed',
                'filename': 'logs/error/error.log',
                'when': 'midnight',
                'backupCount': 90,
                'encoding': 'utf-8'
            },
            
            # Консоль (только для разработки)
            'console': {
                'class': 'logging.StreamHandler',
                'level': 'INFO',
                'formatter': 'simple',
                'stream': 'ext://sys.stdout'
            }
        },
        
        'loggers': {
            'app': {
                'level': 'DEBUG',
                'handlers': ['app_file', 'error_file'],
                'propagate': False
            },
            'app.access': {
                'level': 'INFO',
                'handlers': ['access_file'],
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
            'handlers': ['console'] if os.getenv('DEBUG') else []
        }
    }
    
    logging.config.dictConfig(LOGGING_CONFIG)
5.2. Многопроцессное логирование
python
import logging
import multiprocessing
from logging.handlers import RotatingFileHandler, QueueHandler, QueueListener
import queue


class MultiprocessLogger:
    """
    Логгер для многопроцессных приложений.
    Использует очередь для безопасной записи из разных процессов.
    """
    
    def __init__(self, log_dir: str = "logs", max_mb: int = 100):
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(exist_ok=True)
        self.max_bytes = max_mb * 1024 * 1024
        
        # Очередь для сообщений от всех процессов
        self.log_queue = multiprocessing.Queue(-1)
        
        # Настройка обработчиков в основном процессе
        self._setup_queue_listener()
    
    def _setup_queue_listener(self):
        """Настройка слушателя очереди."""
        # Обработчик для записи в файл с ротацией
        file_handler = RotatingFileHandler(
            filename=self.log_dir / "app.log",
            maxBytes=self.max_bytes,
            backupCount=5,
            encoding='utf-8'
        )
        file_handler.setFormatter(logging.Formatter(
            '%(asctime)s - %(processName)s - %(levelname)s - %(message)s'
        ))
        
        # Обработчик для ошибок
        error_handler = RotatingFileHandler(
            filename=self.log_dir / "error.log",
            maxBytes=self.max_bytes,
            backupCount=5,
            encoding='utf-8'
        )
        error_handler.setLevel(logging.ERROR)
        error_handler.setFormatter(logging.Formatter(
            '%(asctime)s - %(processName)s - %(levelname)s - %(message)s\n%(exc_info)s'
        ))
        
        # Слушатель очереди
        self.listener = QueueListener(
            self.log_queue,
            file_handler,
            error_handler,
            respect_handler_level=True
        )
    
    def start(self):
        """Запуск слушателя."""
        self.listener.start()
    
    def stop(self):
        """Остановка слушателя."""
        self.listener.stop()
    
    def get_logger(self, name: str = None) -> logging.Logger:
        """Получение логгера для текущего процесса."""
        logger = logging.getLogger(name or __name__)
        
        # Добавляем обработчик очереди
        queue_handler = QueueHandler(self.log_queue)
        logger.addHandler(queue_handler)
        logger.setLevel(logging.DEBUG)
        
        return logger


# Использование
def worker_process(worker_id: int, multiprocess_logger: MultiprocessLogger):
    """Рабочий процесс."""
    logger = multiprocess_logger.get_logger(f"worker.{worker_id}")
    
    for i in range(10):
        logger.info(f"Worker {worker_id} - сообщение {i}")


if __name__ == "__main__":
    mp_logger = MultiprocessLogger()
    mp_logger.start()
    
    processes = []
    for i in range(4):
        p = multiprocessing.Process(target=worker_process, args=(i, mp_logger))
        p.start()
        processes.append(p)
    
    for p in processes:
        p.join()
    
    mp_logger.stop()
6. Практические примеры
6.1. Полная конфигурация для веб-приложения
python
# web_logging.py
import logging
import logging.config
import sys
from pathlib import Path


class WebAppLogging:
    """Полная конфигурация логирования для веб-приложения."""
    
    def __init__(self, app_name: str = "webapp", env: str = "production"):
        self.app_name = app_name
        self.env = env
        self._setup_directories()
        self._configure()
    
    def _setup_directories(self):
        """Создание директорий для логов."""
        dirs = ["logs", "logs/app", "logs/access", "logs/error", "logs/archive"]
        for d in dirs:
            Path(d).mkdir(exist_ok=True)
    
    def _configure(self):
        """Настройка логирования."""
        
        # Базовые настройки
        config = {
            'version': 1,
            'disable_existing_loggers': False,
            
            'formatters': {
                'standard': {
                    'format': '%(asctime)s - %(levelname)s - %(name)s - %(message)s',
                    'datefmt': '%Y-%m-%d %H:%M:%S'
                },
                'detailed': {
                    'format': '%(asctime)s - %(levelname)s - [%(request_id)s] - %(name)s:%(lineno)d - %(message)s',
                    'datefmt': '%Y-%m-%d %H:%M:%S'
                },
                'json': {
                    'format': '{"timestamp":"%(asctime)s","level":"%(levelname)s","logger":"%(name)s","message":"%(message)s"}'
                }
            },
            
            'filters': {
                'request_filter': {
                    '()': 'web_logging.RequestFilter'
                }
            },
            
            'handlers': {
                'app_file': {
                    'class': 'logging.handlers.RotatingFileHandler',
                    'level': 'DEBUG',
                    'formatter': 'detailed',
                    'filename': f'logs/app/{self.app_name}.log',
                    'maxBytes': 50 * 1024 * 1024,
                    'backupCount': 10,
                    'encoding': 'utf-8'
                },
                'access_file': {
                    'class': 'logging.handlers.TimedRotatingFileHandler',
                    'level': 'INFO',
                    'formatter': 'standard',
                    'filename': 'logs/access/access.log',
                    'when': 'midnight',
                    'backupCount': 30,
                    'encoding': 'utf-8'
                },
                'error_file': {
                    'class': 'logging.handlers.TimedRotatingFileHandler',
                    'level': 'ERROR',
                    'formatter': 'detailed',
                    'filename': 'logs/error/error.log',
                    'when': 'midnight',
                    'backupCount': 90,
                    'encoding': 'utf-8'
                }
            }
        }
        
        # Добавляем консоль для разработки
        if self.env != 'production':
            config['handlers']['console'] = {
                'class': 'logging.StreamHandler',
                'level': 'DEBUG',
                'formatter': 'standard',
                'stream': sys.stdout
            }
        
        logging.config.dictConfig(config)
    
    def get_logger(self, name: str) -> logging.Logger:
        """Получение логгера."""
        return logging.getLogger(name)


# Использование
logging_setup = WebAppLogging(app_name="myapp", env="production")
logger = logging_setup.get_logger("app.views")
6.2. Мониторинг ротации логов
python
import os
import time
from pathlib import Path
from datetime import datetime


class LogRotationMonitor:
    """Мониторинг состояния ротации логов."""
    
    def __init__(self, log_dir: str = "logs"):
        self.log_dir = Path(log_dir)
    
    def get_log_stats(self) -> dict:
        """Получение статистики по логам."""
        stats = {}
        
        for log_file in self.log_dir.rglob("*.log*"):
            stat = log_file.stat()
            stats[log_file.name] = {
                'size_mb': stat.st_size / (1024 * 1024),
                'modified': datetime.fromtimestamp(stat.st_mtime),
                'is_compressed': log_file.suffix == '.gz'
            }
        
        return stats
    
    def check_rotation_status(self) -> dict:
        """Проверка статуса ротации."""
        status = {
            'total_files': 0,
            'total_size_mb': 0,
            'oldest_file': None,
            'newest_file': None,
            'warnings': []
        }
        
        for log_file in self.log_dir.rglob("*.log*"):
            stat = log_file.stat()
            size_mb = stat.st_size / (1024 * 1024)
            mtime = datetime.fromtimestamp(stat.st_mtime)
            
            status['total_files'] += 1
            status['total_size_mb'] += size_mb
            
            if status['oldest_file'] is None or mtime < status['oldest_file'][1]:
                status['oldest_file'] = (log_file.name, mtime)
            if status['newest_file'] is None or mtime > status['newest_file'][1]:
                status['newest_file'] = (log_file.name, mtime)
            
            # Предупреждение о больших файлах
            if size_mb > 100:
                status['warnings'].append(f"Файл {log_file.name} слишком большой: {size_mb:.1f} MB")
        
        status['total_size_mb'] = round(status['total_size_mb'], 2)
        return status
    
    def cleanup_old_logs(self, days: int = 30):
        """Очистка старых логов."""
        cutoff = datetime.now().timestamp() - (days * 24 * 3600)
        removed = []
        
        for log_file in self.log_dir.rglob("*.log*"):
            if log_file.stat().st_mtime < cutoff:
                log_file.unlink()
                removed.append(log_file.name)
        
        return removed
7. Контрольные вопросы
В чём разница между RotatingFileHandler и TimedRotatingFileHandler?

Как настроить ротацию логов каждый час?

Что произойдёт, если файл лога достигнет maxBytes?

Как сжимать старые лог-файлы?

Как организовать многопроцессное логирование с ротацией?

Какие проблемы могут возникнуть при ротации логов в многопроцессном приложении?

Как настроить ежедневную ротацию с хранением 30 дней?

Как ограничить общий размер всех лог-файлов?

Как мониторить состояние ротации логов?

Как настроить ротацию для access-логов отдельно от error-логов?

8. Практическое задание
Задание 1 (базовое)
Настройте ротацию логов по размеру: файл 1 MB, хранить 3 копии. Напишите программу, генерирующую много логов, и проверьте ротацию.

Задание 2 (среднее)
Настройте ежедневную ротацию логов с хранением 7 дней. Создайте логгер для разных модулей приложения.

Задание 3 (сложное)
Создайте систему логирования для микросервиса:

Отдельные лог-файлы для каждого сервиса

Ротация по размеру (50 MB) и по времени (ежедневно)

Сжатие старых логов

Мониторинг размера логов

Автоматическая очистка логов старше 30 дней

9. Шпаргалка
python
# === РОТАЦИЯ ПО РАЗМЕРУ ===
from logging.handlers import RotatingFileHandler

handler = RotatingFileHandler(
    'app.log',
    maxBytes=10_000_000,  # 10 MB
    backupCount=5
)

# === РОТАЦИЯ ПО ВРЕМЕНИ ===
from logging.handlers import TimedRotatingFileHandler

handler = TimedRotatingFileHandler(
    'app.log',
    when='midnight',
    interval=1,
    backupCount=30
)

# === ДОПОЛНИТЕЛЬНЫЕ ПАРАМЕТРЫ ===
handler.suffix = "%Y-%m-%d"  # формат суффикса
handler.encoding = 'utf-8'
handler.namer = lambda name: name + ".gz"  # сжатие

# === МНОГОПРОЦЕССНОЕ ЛОГИРОВАНИЕ ===
from logging.handlers import QueueHandler, QueueListener

queue = multiprocessing.Queue()
queue_handler = QueueHandler(queue)
listener = QueueListener(queue, *handlers)
Итог лекции
Вы сегодня:

✅ Изучили механизмы ротации логов (по размеру и по времени)

✅ Освоили настройку RotatingFileHandler и TimedRotatingFileHandler

✅ Научились сжимать и архивировать старые логи

✅ Реализовали многопроцессное логирование с ротацией

✅ Настроили полную систему логирования для production-проекта

Теперь ваши логи никогда не заполнят диск и всегда будут под рукой для анализа!
