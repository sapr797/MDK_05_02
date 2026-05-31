# Тема 2.31. Практические кейсы ETL+валидация. Трансформация данных. Обработка ошибок в ETL-процессах. Разработка ETL-скрипта с трансформацией. Отладка сложных ошибок

**Цель лекции:**  
Изучить практические кейсы разработки ETL-скриптов с валидацией и трансформацией данных, освоить методы обработки ошибок, научиться отлаживать сложные проблемы в ETL-процессах.

> Главная мысль: **Настоящие данные никогда не бывают идеальными. Хороший ETL-скрипт должен не только обрабатывать чистые данные, но и корректно реагировать на грязь, пропуски и ошибки.**

---

## Содержание

1. [Введение в практические ETL-кейсы](#1-введение-в-практические-etl-кейсы)
2. [Трансформация данных](#2-трансформация-данных)
3. [Обработка ошибок в ETL](#3-обработка-ошибок-в-etl)
4. [Отладка сложных ошибок](#4-отладка-сложных-ошибок)
5. [Практические кейсы](#5-практические-кейсы)
6. [Контрольные вопросы](#6-контрольные-вопросы)
7. [Практическое задание](#7-практическое-задание)
8. [Шпаргалка](#8-шпаргалка)

---

## 1. Введение в практические ETL-кейсы

### 1.1. Реальные сценарии ETL

| Сценарий | Источник | Трансформация | Назначение |
|----------|----------|---------------|------------|
| **Агрегация логов** | Логи сервера (текст) | Парсинг, группировка по часам | БД для аналитики |
| **Объединение CRM и ERP** | CSV из CRM, JSON из ERP | Нормализация полей, дедупликация | Data Warehouse |
| **Валидация пользователей** | API регистрации | Проверка email, телефона, возраста | База пользователей |
| **Финансовый отчёт** | Выписки из банка | Конвертация валют, расчёт налогов | Excel отчёт |
| **Синхронизация каталога** | XML фид поставщика | Маппинг категорий, обогащение | База товаров |

### 1.2. Архитектура ETL-скрипта
┌─────────────────────────────────────────────────────────────────────────────┐
│ АРХИТЕКТУРА ETL-СКРИПТА │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ ОСНОВНЫЕ КОМПОНЕНТЫ │ │
│ │ │ │
│ │ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │ │
│ │ │ EXTRACT │───►│ VALIDATE │───►│ TRANSFORM │ │ │
│ │ │ │ │ │ │ │ │ │
│ │ │ - Чтение │ │ - Схема │ │ - Очистка │ │ │
│ │ │ - Парсинг │ │ - Типы │ │ - Нормализ. │ │ │
│ │ │ - Загрузка │ │ - Бизнес- │ │ - Агрегация │ │ │
│ │ │ │ │ правила │ │ - Обогащение │ │ │
│ │ └──────────────┘ └──────────────┘ └──────┬───────┘ │ │
│ │ │ │ │
│ │ ▼ │ │
│ │ ┌──────────────┐ │ │
│ │ │ LOAD │ │ │
│ │ │ │ │ │
│ │ │ - Запись │ │ │
│ │ │ - Сохранение │ │ │
│ │ │ - Commit │ │ │
│ │ └──────────────┘ │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ СКВОЗНЫЕ КОМПОНЕНТЫ │ │
│ │ │ │
│ │ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │ │
│ │ │ Логирование │ │ Метрики │ │ Отчёты об │ │ │
│ │ │ │ │ │ │ ошибках │ │ │
│ │ └──────────────┘ └──────────────┘ └──────────────┘ │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

---

## 2. Трансформация данных

### 2.1. Типовые трансформации

| Тип трансформации | Описание | Пример |
|-------------------|----------|--------|
| **Очистка** | Удаление пробелов, спецсимволов | `"  Иван  "` → `"Иван"` |
| **Нормализация** | Приведение к единому формату | `"Иванов Иван"` → `"Иванов И.`" |
| **Маппинг** | Преобразование значений | `"M"` → `"Мужской"` |
| **Агрегация** | Группировка и суммирование | `sum(продажи) по месяцам` |
| **Фильтрация** | Отбор нужных записей | `age >= 18` |
| **Обогащение** | Добавление данных из других источников | Добавление геолокации по IP |
| **Дедупликация** | Удаление дубликатов | Удаление одинаковых email |
| **Расчёт** | Вычисление новых полей | `total = price * quantity` |

### 2.2. Базовые трансформации

```python
from typing import Dict, Any, List, Optional
from datetime import datetime
import re


class DataTransformer:
    """Набор базовых трансформаций для ETL-процессов."""
    
    # ===== ОЧИСТКА =====
    
    @staticmethod
    def trim(data: List[Dict], fields: List[str]) -> List[Dict]:
        """Удаление пробелов в начале и конце строк."""
        for row in data:
            for field in fields:
                if field in row and isinstance(row[field], str):
                    row[field] = row[field].strip()
        return data
    
    @staticmethod
    def remove_null_values(data: List[Dict], fields: List[str]) -> List[Dict]:
        """Замена None на пустую строку или значение по умолчанию."""
        for row in data:
            for field in fields:
                if row.get(field) is None:
                    row[field] = ""
        return data
    
    # ===== НОРМАЛИЗАЦИЯ =====
    
    @staticmethod
    def normalize_case(data: List[Dict], fields: List[str], case: str = "lower") -> List[Dict]:
        """Нормализация регистра."""
        for row in data:
            for field in fields:
                if field in row and isinstance(row[field], str):
                    if case == "lower":
                        row[field] = row[field].lower()
                    elif case == "upper":
                        row[field] = row[field].upper()
                    elif case == "title":
                        row[field] = row[field].title()
        return data
    
    @staticmethod
    def normalize_phone(phone: str) -> Optional[str]:
        """Нормализация российского номера телефона."""
        if not phone:
            return None
        
        # Удаляем всё, кроме цифр
        digits = re.sub(r'\D', '', phone)
        
        if len(digits) == 11 and digits[0] == '8':
            digits = '7' + digits[1:]
        elif len(digits) == 10:
            digits = '7' + digits
        
        if len(digits) == 11 and digits[0] == '7':
            return f"+{digits}"
        
        return None
    
    # ===== МАППИНГ =====
    
    @staticmethod
    def map_values(data: List[Dict], field: str, mapping: Dict) -> List[Dict]:
        """Маппинг значений."""
        for row in data:
            if field in row and row[field] in mapping:
                row[field] = mapping[row[field]]
        return data
    
    # ===== ФИЛЬТРАЦИЯ =====
    
    @staticmethod
    def filter_by_condition(data: List[Dict], condition_func) -> List[Dict]:
        """Фильтрация по условию."""
        return [row for row in data if condition_func(row)]
    
    @staticmethod
    def filter_out_null(data: List[Dict], fields: List[str]) -> List[Dict]:
        """Удаление записей с null в указанных полях."""
        return [
            row for row in data
            if all(row.get(field) is not None for field in fields)
        ]
    
    # ===== ДЕДУПЛИКАЦИЯ =====
    
    @staticmethod
    def deduplicate(data: List[Dict], key_fields: List[str]) -> List[Dict]:
        """Удаление дубликатов по ключевым полям."""
        seen = set()
        unique_data = []
        
        for row in data:
            key = tuple(row.get(field) for field in key_fields)
            if key not in seen:
                seen.add(key)
                unique_data.append(row)
        
        return unique_data
    
    # ===== РАСЧЁТ =====
    
    @staticmethod
    def add_calculated_field(
        data: List[Dict],
        new_field: str,
        calculation_func
    ) -> List[Dict]:
        """Добавление вычисляемого поля."""
        for row in data:
            row[new_field] = calculation_func(row)
        return data
    
    @staticmethod
    def calculate_age(birth_date: datetime) -> int:
        """Расчёт возраста по дате рождения."""
        today = datetime.now()
        age = today.year - birth_date.year
        if (today.month, today.day) < (birth_date.month, birth_date.day):
            age -= 1
        return age
    
    # ===== АГРЕГАЦИЯ =====
    
    @staticmethod
    def aggregate(
        data: List[Dict],
        group_by: List[str],
        aggregations: Dict[str, str]
    ) -> List[Dict]:
        """
        Агрегация данных.
        
        Args:
            group_by: Список полей для группировки
            aggregations: Словарь {поле: функция} (sum, avg, count, min, max)
        """
        groups = {}
        
        for row in data:
            key = tuple(row.get(field) for field in group_by)
            
            if key not in groups:
                groups[key] = {field: row.get(field) for field in group_by}
                groups[key]['_count'] = 0
                for agg_field in aggregations:
                    groups[key][f"{agg_field}_sum"] = 0
                    groups[key][f"{agg_field}_min"] = None
                    groups[key][f"{agg_field}_max"] = None
                    groups[key][f"{agg_field}_values"] = []
            
            groups[key]['_count'] += 1
            
            for agg_field, func in aggregations.items():
                value = row.get(agg_field, 0)
                if func in ['sum', 'avg']:
                    groups[key][f"{agg_field}_sum"] += value
                    groups[key][f"{agg_field}_values"].append(value)
                elif func == 'min':
                    current_min = groups[key][f"{agg_field}_min"]
                    if current_min is None or value < current_min:
                        groups[key][f"{agg_field}_min"] = value
                elif func == 'max':
                    current_max = groups[key][f"{agg_field}_max"]
                    if current_max is None or value > current_max:
                        groups[key][f"{agg_field}_max"] = value
        
        # Постобработка для средних значений
        result = []
        for key, group in groups.items():
            for agg_field, func in aggregations.items():
                if func == 'avg':
                    group[f"{agg_field}_avg"] = group[f"{agg_field}_sum"] / group['_count']
                del group[f"{agg_field}_values"]
            
            result.append(group)
        
        return result
3. Обработка ошибок в ETL
3.1. Типы ошибок в ETL
Тип ошибки	Описание	Стратегия обработки
Ошибка подключения	Источник недоступен	Retry с экспоненциальной задержкой
Ошибка формата	Неверный формат данных	Пропуск строки, логирование
Ошибка валидации	Данные не проходят проверку	Отбраковка в отдельный файл
Ошибка трансформации	Ошибка в преобразовании	Логирование, пропуск записи
Ошибка загрузки	Целевая система недоступна	Retry, fallback
3.2. Механизмы обработки ошибок
python
import time
import logging
from functools import wraps
from typing import Callable, Any, Optional
from datetime import datetime


class ErrorHandler:
    """Централизованная обработка ошибок в ETL."""
    
    def __init__(self, error_log_path: str = "errors.json"):
        self.error_log_path = error_log_path
        self.errors = []
        self.setup_logging()
    
    def setup_logging(self):
        """Настройка логирования."""
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s',
            handlers=[
                logging.FileHandler('etl.log'),
                logging.StreamHandler()
            ]
        )
        self.logger = logging.getLogger(__name__)
    
    def log_error(self, error: Exception, context: dict = None):
        """Логирование ошибки."""
        error_record = {
            "timestamp": datetime.now().isoformat(),
            "error_type": type(error).__name__,
            "message": str(error),
            "context": context or {}
        }
        self.errors.append(error_record)
        self.logger.error(f"{error_record['error_type']}: {error_record['message']}")
        
        # Сохранение в файл
        import json
        with open(self.error_log_path, 'w', encoding='utf-8') as f:
            json.dump(self.errors, f, indent=2, ensure_ascii=False)
    
    def get_errors(self) -> list:
        """Получение списка ошибок."""
        return self.errors


class RetryDecorator:
    """Декоратор для повторных попыток."""
    
    def __init__(
        self,
        max_retries: int = 3,
        delay: float = 1.0,
        backoff: float = 2.0,
        exceptions: tuple = (Exception,)
    ):
        self.max_retries = max_retries
        self.delay = delay
        self.backoff = backoff
        self.exceptions = exceptions
    
    def __call__(self, func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            current_delay = self.delay
            
            for attempt in range(1, self.max_retries + 1):
                try:
                    return func(*args, **kwargs)
                except self.exceptions as e:
                    last_exception = e
                    if attempt == self.max_retries:
                        break
                    
                    print(f"Попытка {attempt} не удалась. Повтор через {current_delay} сек...")
                    time.sleep(current_delay)
                    current_delay *= self.backoff
            
            raise last_exception
        
        return wrapper


@RetryDecorator(max_retries=3, delay=1.0, exceptions=(ConnectionError, TimeoutError))
def fetch_from_api(url: str) -> dict:
    """Загрузка данных с API с повторными попытками."""
    import requests
    response = requests.get(url, timeout=5)
    response.raise_for_status()
    return response.json()
3.3. Dead Letter Queue (DLQ)
python
class DeadLetterQueue:
    """Очередь ошибочных записей."""
    
    def __init__(self, dlq_path: str = "dead_letter_queue"):
        self.dlq_path = Path(dlq_path)
        self.dlq_path.mkdir(exist_ok=True)
    
    def send_to_dlq(self, record: dict, error: str, stage: str):
        """Отправка записи в DLQ."""
        timestamp = datetime.now().strftime("%Y%m%d")
        dlq_file = self.dlq_path / f"{stage}_{timestamp}.json"
        
        error_record = {
            "original_record": record,
            "error": error,
            "stage": stage,
            "timestamp": datetime.now().isoformat()
        }
        
        # Чтение существующих
        existing = []
        if dlq_file.exists():
            import json
            with open(dlq_file, 'r', encoding='utf-8') as f:
                existing = json.load(f)
        
        existing.append(error_record)
        
        # Запись
        import json
        with open(dlq_file, 'w', encoding='utf-8') as f:
            json.dump(existing, f, indent=2, ensure_ascii=False)
        
        print(f"📦 Запись отправлена в DLQ: {dlq_file}")
    
    def replay(self, stage: str, callback: Callable):
        """Повторная обработка записей из DLQ."""
        import json
        
        for dlq_file in self.dlq_path.glob(f"{stage}_*.json"):
            with open(dlq_file, 'r', encoding='utf-8') as f:
                records = json.load(f)
            
            for record in records:
                try:
                    callback(record['original_record'])
                    print(f"✅ Запись успешно обработана повторно")
                except Exception as e:
                    print(f"❌ Повторная обработка не удалась: {e}")
4. Отладка сложных ошибок
4.1. Инструменты отладки ETL
Инструмент	Назначение	Когда использовать
Логирование	Запись хода выполнения	Всегда
Точки останова	Интерактивная отладка	При сложных ошибках
Тестовые данные	Воспроизведение ошибки	Для изоляции проблемы
Профилирование	Поиск узких мест	При проблемах с производительностью
Визуализация	Понимание потока данных	При сложных трансформациях
4.2. Техники отладки
python
class ETLDebugger:
    """Инструменты для отладки ETL-процессов."""
    
    def __init__(self, debug_mode: bool = False):
        self.debug_mode = debug_mode
        self.checkpoints = []
    
    def checkpoint(self, name: str, data: Any, stage: str) -> Any:
        """Сохранение контрольной точки."""
        if self.debug_mode:
            checkpoint_data = {
                "name": name,
                "stage": stage,
                "timestamp": datetime.now().isoformat(),
                "data_type": type(data).__name__,
                "data_length": len(data) if hasattr(data, '__len__') else None
            }
            self.checkpoints.append(checkpoint_data)
            
            # Сохранение среза данных для анализа
            data_sample = data[:5] if isinstance(data, list) and len(data) > 5 else data
            print(f"\n🔍 КОНТРОЛЬНАЯ ТОЧКА: {name}")
            print(f"   Этап: {stage}")
            print(f"   Всего записей: {len(data) if hasattr(data, '__len__') else 'N/A'}")
            print(f"   Пример данных: {data_sample}")
        
        return data
    
    def inspect_row(self, row: dict, row_index: int):
        """Детальный анализ строки."""
        print(f"\n🔬 ИНСПЕКЦИЯ СТРОКИ #{row_index}")
        print("-" * 40)
        for key, value in row.items():
            print(f"   {key}: {repr(value)} ({type(value).__name__})")
        print("-" * 40)
    
    def compare_dataframes(self, before: list, after: list, name: str):
        """Сравнение данных до и после трансформации."""
        print(f"\n📊 СРАВНЕНИЕ ДАННЫХ: {name}")
        print(f"   Было записей: {len(before)}")
        print(f"   Стало записей: {len(after)}")
        
        if len(before) != len(after):
            print(f"   ⚠️ ИЗМЕНЕНИЕ КОЛИЧЕСТВА ЗАПИСЕЙ: {len(after) - len(before):+d}")
    
    def trace_through_transformations(self, data: list, transformations: list):
        """Трассировка данных через последовательность трансформаций."""
        current_data = data
        
        for i, transform in enumerate(transformations):
            transform_name = transform.__name__ if hasattr(transform, '__name__') else str(transform)
            current_data = self.checkpoint(
                f"После {transform_name}",
                current_data,
                f"transform_{i}"
            )
        
        return current_data
    
    def sample_by_condition(self, data: list, condition_func, limit: int = 5) -> list:
        """Выборка записей по условию."""
        samples = []
        for row in data:
            if condition_func(row):
                samples.append(row)
                if len(samples) >= limit:
                    break
        return samples
    
    def get_report(self) -> dict:
        """Отчёт об отладке."""
        return {
            "debug_mode": self.debug_mode,
            "checkpoints_count": len(self.checkpoints),
            "checkpoints": self.checkpoints
        }


def debug_on_error(func):
    """Декоратор для отладки при ошибке."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            print(f"\n🔥 ОШИБКА В {func.__name__}: {e}")
            print(f"   Аргументы: {args}")
            print(f"   Ключевые аргументы: {kwargs}")
            
            # Сохранение контекста ошибки
            import traceback
            traceback.print_exc()
            
            raise
    
    return wrapper
4.3. Профилирование ETL-скриптов
python
import time
from functools import wraps
from collections import defaultdict


class PerformanceProfiler:
    """Профилировщик производительности ETL-процессов."""
    
    def __init__(self):
        self.metrics = defaultdict(list)
    
    def profile(self, name: str):
        """Декоратор для профилирования функций."""
        def decorator(func):
            @wraps(func)
            def wrapper(*args, **kwargs):
                start = time.time()
                result = func(*args, **kwargs)
                elapsed = time.time() - start
                self.metrics[name].append(elapsed)
                print(f"⏱️ {name}: {elapsed:.3f} сек")
                return result
            return wrapper
        return decorator
    
    def get_report(self) -> dict:
        """Отчёт о производительности."""
        report = {}
        for name, times in self.metrics.items():
            report[name] = {
                "count": len(times),
                "total_time": sum(times),
                "avg_time": sum(times) / len(times) if times else 0,
                "min_time": min(times) if times else 0,
                "max_time": max(times) if times else 0
            }
        return report
    
    def print_report(self):
        """Вывод отчёта."""
        print("\n" + "=" * 50)
        print("ОТЧЁТ О ПРОИЗВОДИТЕЛЬНОСТИ")
        print("=" * 50)
        for name, stats in self.get_report().items():
            print(f"\n{name}:")
            print(f"   Вызовов: {stats['count']}")
            print(f"   Общее время: {stats['total_time']:.3f} сек")
            print(f"   Среднее: {stats['avg_time']:.3f} сек")
            print(f"   Минимум: {stats['min_time']:.3f} сек")
            print(f"   Максимум: {stats['max_time']:.3f} сек")
5. Практические кейсы
5.1. Кейс 1: Обработка логов сервера
python
"""
Кейс 1: ETL-скрипт для обработки логов веб-сервера.
Задача: Парсинг access.log, агрегация по часам, выявление подозрительной активности.
"""

import re
from datetime import datetime
from collections import defaultdict
from typing import List, Dict, Any


class LogProcessor:
    """Обработчик логов веб-сервера."""
    
    # Паттерн для парсинга access.log
    LOG_PATTERN = re.compile(
        r'(?P<ip>\S+) \S+ \S+ \[(?P<time>.*?)\] '
        r'"(?P<method>\S+) (?P<url>\S+) \S+" '
        r'(?P<status>\d+) (?P<size>\d+) '
        r'"(?P<referer>.*?)" "(?P<user_agent>.*?)"'
    )
    
    def __init__(self):
        self.raw_data = []
        self.parsed_data = []
        self.error_handler = ErrorHandler()
    
    def extract(self, log_file: str) -> List[str]:
        """Извлечение строк логов."""
        print(f"📂 Чтение логов из {log_file}")
        
        with open(log_file, 'r', encoding='utf-8') as f:
            lines = f.readlines()
        
        print(f"   Прочитано строк: {len(lines)}")
        return lines
    
    def parse_line(self, line: str, line_num: int) -> Dict:
        """Парсинг одной строки лога."""
        match = self.LOG_PATTERN.match(line.strip())
        
        if not match:
            self.error_handler.log_error(
                ValueError(f"Не удалось распарсить строку"),
                {"line_num": line_num, "content": line[:100]}
            )
            return None
        
        data = match.groupdict()
        
        # Преобразование типов
        data['status'] = int(data['status'])
        data['size'] = int(data['size']) if data['size'] != '-' else 0
        
        # Парсинг времени
        try:
            data['timestamp'] = datetime.strptime(
                data['time'].split()[0],
                "%d/%b/%Y:%H:%M:%S"
            )
        except ValueError:
            data['timestamp'] = None
        
        return data
    
    def transform(self, data: List[str]) -> List[Dict]:
        """Трансформация данных."""
        print("🔄 Трансформация данных...")
        
        parsed = []
        for i, line in enumerate(data):
            parsed_line = self.parse_line(line, i)
            if parsed_line:
                parsed.append(parsed_line)
        
        print(f"   Успешно распарсено: {len(parsed)}")
        print(f"   Ошибок: {len(self.error_handler.get_errors())}")
        return parsed
    
    def aggregate_by_hour(self, data: List[Dict]) -> List[Dict]:
        """Агрегация по часам."""
        hourly_stats = defaultdict(lambda: {
            'requests': 0,
            'bytes_total': 0,
            'status_2xx': 0,
            'status_3xx': 0,
            'status_4xx': 0,
            'status_5xx': 0,
            'unique_ips': set(),
            'top_urls': defaultdict(int)
        })
        
        for record in data:
            if not record.get('timestamp'):
                continue
            
            hour = record['timestamp'].strftime("%Y-%m-%d %H:00:00")
            stats = hourly_stats[hour]
            
            stats['requests'] += 1
            stats['bytes_total'] += record.get('size', 0)
            
            status = record.get('status', 0)
            if 200 <= status < 300:
                stats['status_2xx'] += 1
            elif 300 <= status < 400:
                stats['status_3xx'] += 1
            elif 400 <= status < 500:
                stats['status_4xx'] += 1
            elif 500 <= status < 600:
                stats['status_5xx'] += 1
            
            stats['unique_ips'].add(record.get('ip'))
            stats['top_urls'][record.get('url', '')] += 1
        
        # Преобразование в список
        result = []
        for hour, stats in hourly_stats.items():
            result.append({
                'hour': hour,
                'requests': stats['requests'],
                'bytes_total': stats['bytes_total'],
                'status_2xx': stats['status_2xx'],
                'status_3xx': stats['status_3xx'],
                'status_4xx': stats['status_4xx'],
                'status_5xx': stats['status_5xx'],
                'unique_ips': len(stats['unique_ips']),
                'top_url': max(stats['top_urls'].items(), key=lambda x: x[1])[0] if stats['top_urls'] else None
            })
        
        return sorted(result, key=lambda x: x['hour'])
    
    def detect_suspicious_activity(self, data: List[Dict]) -> List[Dict]:
        """Выявление подозрительной активности."""
        suspicious = []
        
        # Подозрительные статусы
        for record in data:
            if record.get('status') in [401, 403, 404, 500, 502, 503]:
                suspicious.append({
                    'timestamp': record.get('timestamp'),
                    'ip': record.get('ip'),
                    'url': record.get('url'),
                    'status': record.get('status'),
                    'reason': f"Подозрительный статус {record.get('status')}"
                })
        
        # Группировка по IP для выявления атак
        ip_requests = defaultdict(int)
        ip_404 = defaultdict(int)
        
        for record in data:
            ip_requests[record.get('ip')] += 1
            if record.get('status') == 404:
                ip_404[record.get('ip')] += 1
        
        for ip, count in ip_404.items():
            if count > 10 and count / ip_requests[ip] > 0.5:
                suspicious.append({
                    'timestamp': None,
                    'ip': ip,
                    'status': None,
                    'reason': f"Сканирование: {count} ошибок 404 из {ip_requests[ip]} запросов"
                })
        
        return suspicious
    
    def load(self, data: List[Dict], output_file: str):
        """Загрузка результатов."""
        import json
        from datetime import datetime
        
        output = {
            "generated_at": datetime.now().isoformat(),
            "total_requests": len(data),
            "hourly_stats": self.aggregate_by_hour(data),
            "suspicious_activity": self.detect_suspicious_activity(data)
        }
        
        with open(output_file, 'w', encoding='utf-8') as f:
            json.dump(output, f, indent=2, ensure_ascii=False, default=str)
        
        print(f"💾 Результаты сохранены в {output_file}")
    
    def run(self, log_file: str, output_file: str):
        """Запуск ETL-процесса."""
        print("=" * 60)
        print("ETL-ОБРАБОТКА ЛОГОВ ВЕБ-СЕРВЕРА")
        print("=" * 60)
        
        # EXTRACT
        raw_data = self.extract(log_file)
        
        # TRANSFORM
        parsed_data = self.transform(raw_data)
        
        # LOAD
        self.load(parsed_data, output_file)
        
        # Отчёт об ошибках
        errors = self.error_handler.get_errors()
        if errors:
            print(f"\n⚠️ Зафиксировано ошибок: {len(errors)}")
        
        print("\n✅ ETL-процесс завершён")
5.2. Кейс 2: Обработка пользовательских данных
python
"""
Кейс 2: ETL-скрипт для обработки пользовательских данных.
Задача: Чтение CSV, валидация, обогащение, загрузка в JSON.
"""

import csv
import json
from datetime import datetime
from typing import List, Dict, Any, Optional
from pathlib import Path


class UserDataProcessor:
    """Обработчик пользовательских данных."""
    
    def __init__(self):
        self.stats = {
            "total": 0,
            "valid": 0,
            "invalid": 0,
            "errors": []
        }
    
    def extract(self, csv_file: str) -> List[Dict]:
        """Извлечение данных из CSV."""
        print(f"📂 Чтение CSV из {csv_file}")
        
        data = []
        with open(csv_file, 'r', encoding='utf-8') as f:
            reader = csv.DictReader(f)
            for row in reader:
                # Преобразование типов
                if 'age' in row and row['age']:
                    try:
                        row['age'] = int(row['age'])
                    except ValueError:
                        pass
                data.append(row)
        
        self.stats["total"] = len(data)
        print(f"   Прочитано записей: {len(data)}")
        return data
    
    def validate_email(self, email: str) -> bool:
        """Валидация email."""
        if not email:
            return False
        return '@' in email and '.' in email
    
    def validate_phone(self, phone: str) -> Optional[str]:
        """Валидация и нормализация телефона."""
        if not phone:
            return None
        
        import re
        digits = re.sub(r'\D', '', phone)
        
        if len(digits) == 11 and digits[0] == '8':
            digits = '7' + digits[1:]
        
        if len(digits) == 11 and digits[0] == '7':
            return f"+{digits}"
        
        return None
    
    def validate(self, data: List[Dict]) -> List[Dict]:
        """Валидация данных."""
        print("🔍 Валидация данных...")
        
        valid_data = []
        
        for i, row in enumerate(data):
            errors = []
            
            # Проверка имени
            if not row.get('name'):
                errors.append("Имя обязательно")
            
            # Проверка email
            if not self.validate_email(row.get('email', '')):
                errors.append("Неверный формат email")
            
            # Проверка возраста
            age = row.get('age')
            if age is not None:
                try:
                    age_int = int(age)
                    if age_int < 0 or age_int > 150:
                        errors.append("Некорректный возраст")
                except (ValueError, TypeError):
                    errors.append("Возраст должен быть числом")
            
            # Нормализация телефона
            phone = self.validate_phone(row.get('phone', ''))
            if phone:
                row['phone'] = phone
            
            if errors:
                self.stats["invalid"] += 1
                self.stats["errors"].append({
                    "row": i,
                    "data": row,
                    "errors": errors
                })
            else:
                self.stats["valid"] += 1
                valid_data.append(row)
        
        print(f"   Валидных: {self.stats['valid']}")
        print(f"   Невалидных: {self.stats['invalid']}")
        
        return valid_data
    
    def transform(self, data: List[Dict]) -> List[Dict]:
        """Трансформация данных."""
        print("🔄 Трансформация данных...")
        
        for row in data:
            # Нормализация имени
            if row.get('name'):
                row['name'] = row['name'].strip().title()
            
            # Нормализация email
            if row.get('email'):
                row['email'] = row['email'].lower().strip()
            
            # Расчёт возраста по дате рождения (если есть)
            if row.get('birth_date'):
                try:
                    birth_date = datetime.strptime(row['birth_date'], "%Y-%m-%d")
                    today = datetime.now()
                    age = today.year - birth_date.year
                    if (today.month, today.day) < (birth_date.month, birth_date.day):
                        age -= 1
                    row['calculated_age'] = age
                except ValueError:
                    pass
            
            # Добавление временной метки
            row['processed_at'] = datetime.now().isoformat()
        
        print(f"   Трансформировано записей: {len(data)}")
        return data
    
    def load(self, data: List[Dict], output_file: str):
        """Загрузка результатов."""
        output = {
            "statistics": {
                "total_processed": self.stats["total"],
                "valid_records": self.stats["valid"],
                "invalid_records": self.stats["invalid"],
                "success_rate": f"{self.stats['valid'] / self.stats['total'] * 100:.1f}%" if self.stats['total'] else "0%"
            },
            "data": data,
            "validation_errors": self.stats["errors"][:100]  # Только первые 100
        }
        
        with open(output_file, 'w', encoding='utf-8') as f:
            json.dump(output, f, indent=2, ensure_ascii=False, default=str)
        
        print(f"💾 Результаты сохранены в {output_file}")
        
        # Сохранение ошибочных записей отдельно
        if self.stats["errors"]:
            error_file = output_file.replace('.json', '_errors.json')
            with open(error_file, 'w', encoding='utf-8') as f:
                json.dump(self.stats["errors"], f, indent=2, ensure_ascii=False)
            print(f"📝 Ошибки валидации сохранены в {error_file}")
    
    def run(self, input_file: str, output_file: str):
        """Запуск ETL-процесса."""
        print("=" * 60)
        print("ETL-ОБРАБОТКА ПОЛЬЗОВАТЕЛЬСКИХ ДАННЫХ")
        print("=" * 60)
        
        # EXTRACT
        raw_data = self.extract(input_file)
        
        # VALIDATE
        validated_data = self.validate(raw_data)
        
        # TRANSFORM
        transformed_data = self.transform(validated_data)
        
        # LOAD
        self.load(transformed_data, output_file)
        
        print("\n✅ ETL-процесс завершён")
        print(f"📊 Статистика: {self.stats['valid']}/{self.stats['total']} записей прошли валидацию")
6. Контрольные вопросы
Какие основные этапы включает ETL-процесс?

Какие типы трансформаций данных вы знаете?

Как организовать обработку ошибок в ETL-скрипте?

Что такое Dead Letter Queue и зачем она нужна?

Какие инструменты использовать для отладки ETL-процессов?

Как реализовать повторные попытки при временных ошибках?

Как профилировать производительность ETL-скрипта?

Какие данные стоит логировать в ETL-процессе?

Как обрабатывать частичные сбои (несколько записей)?

Как тестировать ETL-скрипты на различных наборах данных?

7. Практическое задание
Задание 1 (базовое)
Разработайте ETL-скрипт для обработки CSV файла с товарами. Добавьте валидацию цен, нормализацию категорий.

Задание 2 (среднее)
Создайте ETL-процесс для агрегации продаж по дням. Реализуйте обработку ошибок и Dead Letter Queue.

Задание 3 (сложное)
Разработайте полноценный ETL-пайплайн для интернет-магазина:

Источники: CSV товары, JSON заказы

Валидация данных (цены, остатки)

Трансформация (расчёт выручки, группировка)

Загрузка в SQLite и JSON отчёт

Логирование и профилирование

8. Шпаргалка
python
# === БАЗОВЫЙ ETL-СКРИПТ ===
def etl_pipeline():
    # Extract
    data = extract(source)
    # Transform
    data = transform(data)
    # Load
    load(data, destination)

# === ОБРАБОТКА ОШИБОК ===
try:
    result = operation()
except SpecificError as e:
    log_error(e)
    send_to_dlq(record, e)
except Exception as e:
    rollback()
    raise

# === RETRY ===
@retry(max_attempts=3, delay=1.0)
def fetch_data():
    return api_call()

# === ПРОФИЛИРОВАНИЕ ===
@profile("extract")
def extract():
    pass

# === ОТЛАДКА ===
debugger = ETLDebugger(debug_mode=True)
debugger.checkpoint("after_extract", data)
Итог лекции
Вы сегодня:

✅ Изучили практические кейсы разработки ETL-скриптов

✅ Освоили различные типы трансформаций данных

✅ Реализовали механизмы обработки ошибок (retry, DLQ)

✅ Изучили инструменты отладки ETL-процессов

✅ Разработали полноценные ETL-скрипты для реальных задач

Теперь вы можете создавать надёжные, отказоустойчивые ETL-системы для любых данных!
