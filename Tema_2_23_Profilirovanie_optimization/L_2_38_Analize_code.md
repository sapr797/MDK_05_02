# Тема 2.38. Анализ узких мест. Оптимизация производительности и медленных участков кода. Кэширование

**Цель лекции:**  
Научиться анализировать производительность Python-кода, выявлять узкие места, применять различные техники оптимизации и использовать кэширование для ускорения работы приложений.

> Главная мысль: **Преждевременная оптимизация — корень всех зол, но осознанная оптимизация на основе данных — признак профессионализма.**

---

## Содержание

1. [Выявление узких мест](#1-выявление-узких-мест)
2. [Стратегии оптимизации](#2-стратегии-оптимизации)
3. [Кэширование результатов](#3-кэширование-результатов)
4. [Оптимизация работы с памятью](#4-оптимизация-работы-с-памятью)
5. [Практические примеры](#5-практические-примеры)
6. [Контрольные вопросы](#6-контрольные-вопросы)
7. [Практическое задание](#7-практическое-задание)
8. [Шпаргалка](#8-шпаргалка)

---

## 1. Выявление узких мест

### 1.1. Признаки узких мест
┌─────────────────────────────────────────────────────────────────────────────┐
│ ПРИЗНАКИ УЗКИХ МЕСТ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ⏱️ ВРЕМЕННЫЕ ПРИЗНАКИ │
│ • Высокое время отклика (>1 сек) │
│ • Медленный запуск приложения │
│ • Задержки при обработке данных │
│ │
│ 💾 РЕСУРСНЫЕ ПРИЗНАКИ │
│ • Высокая загрузка CPU │
│ • Большое потребление памяти │
│ • Частые операции ввода-вывода │
│ │
│ 🔍 ПРОГРАММНЫЕ ПРИЗНАКИ │
│ • Вложенные циклы │
│ • Рекурсивные вызовы │
│ • Частые обращения к БД/API │
│ • Повторные вычисления │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Инструменты для поиска узких мест

```python
import time
import cProfile
import pstats
from functools import wraps
from memory_profiler import profile as mem_profile


def timing_decorator(func):
    """Декоратор для измерения времени выполнения."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__}: {elapsed:.4f} сек")
        return result
    return wrapper


def profile_decorator(output_file=None):
    """Декоратор для cProfile профилирования."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            profiler = cProfile.Profile()
            profiler.enable()
            result = func(*args, **kwargs)
            profiler.disable()
            
            if output_file:
                profiler.dump_stats(output_file)
            else:
                stats = pstats.Stats(profiler)
                stats.sort_stats('cumulative')
                stats.print_stats(20)
            
            return result
        return wrapper
    return decorator


# Пример использования
@timing_decorator
def slow_function():
    total = 0
    for i in range(10_000_000):
        total += i ** 2
    return total


@profile_decorator()
def function_to_profile():
    data = [i ** 2 for i in range(10000)]
    return sum(data)
2. Стратегии оптимизации
2.1. Иерархия оптимизаций
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ПИРАМИДА ОПТИМИЗАЦИИ                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                    ┌─────────────────────┐                                 │
│                    │   Изменение         │                                 │
│                    │   архитектуры       │  ← самый высокий эффект          │
│                    ├─────────────────────┤                                 │
│                    │   Алгоритмическая   │                                 │
│                    │   оптимизация       │                                 │
│                    ├─────────────────────┤                                 │
│                    │   Кэширование       │                                 │
│                    ├─────────────────────┤                                 │
│                    │   Оптимизация       │                                 │
│                    │   циклов            │                                 │
│                    ├─────────────────────┤                                 │
│                    │   Микрооптимизации  │  ← наименьший эффект            │
│                    └─────────────────────┘                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
2.2. Оптимизация циклов
python
import time
from typing import List


class LoopOptimization:
    """Демонстрация оптимизации циклов."""
    
    @staticmethod
    def bad_loop(data: List[int]) -> int:
        """Плохой цикл (медленный)."""
        result = 0
        for i in range(len(data)):
            result += data[i]
        return result
    
    @staticmethod
    def good_loop(data: List[int]) -> int:
        """Хороший цикл (быстрый)."""
        result = 0
        for value in data:
            result += value
        return result
    
    @staticmethod
    def vectorized(data: List[int]) -> int:
        """Векторизованная операция (самый быстрый)."""
        return sum(data)
    
    @staticmethod
    def compare(data_size: int = 10_000_000):
        """Сравнение производительности циклов."""
        data = list(range(data_size))
        
        start = time.perf_counter()
        LoopOptimization.bad_loop(data)
        bad_time = time.perf_counter() - start
        
        start = time.perf_counter()
        LoopOptimization.good_loop(data)
        good_time = time.perf_counter() - start
        
        start = time.perf_counter()
        LoopOptimization.vectorized(data)
        vectorized_time = time.perf_counter() - start
        
        print(f"Плохой цикл: {bad_time:.3f} сек")
        print(f"Хороший цикл: {good_time:.3f} сек")
        print(f"Векторизованный: {vectorized_time:.3f} сек")
2.3. Оптимизация работы со строками
python
class StringOptimization:
    """Оптимизация работы со строками."""
    
    @staticmethod
    def bad_concatenation(words: List[str]) -> str:
        """Плохая конкатенация (медленная)."""
        result = ""
        for word in words:
            result += word + " "
        return result.strip()
    
    @staticmethod
    def good_concatenation(words: List[str]) -> str:
        """Хорошая конкатенация (быстрая)."""
        return " ".join(words)
    
    @staticmethod
    def format_numbers_bad(numbers: List[int]) -> str:
        """Плохое форматирование."""
        result = ""
        for n in numbers:
            result += str(n) + ","
        return result[:-1]
    
    @staticmethod
    def format_numbers_good(numbers: List[int]) -> str:
        """Хорошее форматирование."""
        return ",".join(map(str, numbers))
    
    @staticmethod
    def compare(n: int = 10000):
        """Сравнение производительности."""
        words = [f"word_{i}" for i in range(n)]
        numbers = list(range(n))
        
        start = time.perf_counter()
        StringOptimization.bad_concatenation(words)
        bad_concat = time.perf_counter() - start
        
        start = time.perf_counter()
        StringOptimization.good_concatenation(words)
        good_concat = time.perf_counter() - start
        
        start = time.perf_counter()
        StringOptimization.format_numbers_bad(numbers)
        bad_format = time.perf_counter() - start
        
        start = time.perf_counter()
        StringOptimization.format_numbers_good(numbers)
        good_format = time.perf_counter() - start
        
        print(f"Конкатенация (плохая): {bad_concat:.4f} сек")
        print(f"Конкатенация (хорошая): {good_concat:.4f} сек")
        print(f"Форматирование (плохое): {bad_format:.4f} сек")
        print(f"Форматирование (хорошее): {good_format:.4f} сек")
3. Кэширование результатов
3.1. Использование functools.lru_cache
python
from functools import lru_cache
import time


# Без кэширования (медленно)
def fibonacci_no_cache(n: int) -> int:
    if n < 2:
        return n
    return fibonacci_no_cache(n - 1) + fibonacci_no_cache(n - 2)


# С кэшированием (быстро)
@lru_cache(maxsize=128)
def fibonacci_cached(n: int) -> int:
    if n < 2:
        return n
    return fibonacci_cached(n - 1) + fibonacci_cached(n - 2)


# Пример с пользовательскими типами (хэшируемыми)
@lru_cache(maxsize=1000)
def expensive_computation(x: int, y: int) -> int:
    """Вычисление, которое нужно кэшировать."""
    time.sleep(0.1)  # Имитация тяжёлой работы
    return x ** y


# Очистка кэша
fibonacci_cached.cache_clear()
print(f"Информация о кэше: {fibonacci_cached.cache_info()}")
3.2. Реализация простого кэша
python
from typing import Any, Callable, Dict, Tuple
from datetime import datetime, timedelta
import hashlib
import json


class SimpleCache:
    """Простая реализация кэша в памяти."""
    
    def __init__(self, ttl_seconds: int = 3600):
        self._cache: Dict[str, Tuple[Any, datetime]] = {}
        self.ttl_seconds = ttl_seconds
    
    def _make_key(self, func_name: str, args: tuple, kwargs: dict) -> str:
        """Создание ключа для кэша."""
        key_data = {
            'func': func_name,
            'args': args,
            'kwargs': kwargs
        }
        key_str = json.dumps(key_data, sort_keys=True)
        return hashlib.md5(key_str.encode()).hexdigest()
    
    def get(self, key: str) -> Any:
        """Получение значения из кэша."""
        if key in self._cache:
            value, expires = self._cache[key]
            if datetime.now() < expires:
                return value
            else:
                del self._cache[key]
        return None
    
    def set(self, key: str, value: Any):
        """Сохранение значения в кэш."""
        expires = datetime.now() + timedelta(seconds=self.ttl_seconds)
        self._cache[key] = (value, expires)
    
    def clear(self):
        """Очистка кэша."""
        self._cache.clear()
    
    def __call__(self, func: Callable) -> Callable:
        """Декоратор для кэширования функции."""
        def wrapper(*args, **kwargs):
            key = self._make_key(func.__name__, args, kwargs)
            cached = self.get(key)
            if cached is not None:
                return cached
            result = func(*args, **kwargs)
            self.set(key, result)
            return result
        return wrapper


# Использование
cache = SimpleCache(ttl_seconds=60)

@cache
def slow_function(x: int) -> int:
    time.sleep(1)
    return x ** 2


class TTLDictCache:
    """Кэш с временем жизни для каждого ключа."""
    
    def __init__(self, default_ttl: int = 300):
        self._cache: Dict[str, tuple] = {}
        self.default_ttl = default_ttl
    
    def set(self, key: str, value: Any, ttl: int = None):
        ttl = ttl or self.default_ttl
        expires = time.time() + ttl
        self._cache[key] = (value, expires)
    
    def get(self, key: str) -> Any:
        if key in self._cache:
            value, expires = self._cache[key]
            if time.time() < expires:
                return value
            else:
                del self._cache[key]
        return None
    
    def delete(self, key: str):
        if key in self._cache:
            del self._cache[key]
    
    def clear(self):
        self._cache.clear()
    
    def cleanup(self):
        """Очистка просроченных записей."""
        now = time.time()
        expired = [k for k, (_, exp) in self._cache.items() if now >= exp]
        for k in expired:
            del self._cache[k]
        return len(expired)
3.3. Кэширование результатов API запросов
python
import requests
import json
from pathlib import Path
from datetime import datetime


class FileCache:
    """Кэш на диске для API запросов."""
    
    def __init__(self, cache_dir: str = "api_cache", ttl_seconds: int = 3600):
        self.cache_dir = Path(cache_dir)
        self.cache_dir.mkdir(exist_ok=True)
        self.ttl_seconds = ttl_seconds
    
    def _get_cache_path(self, url: str, params: dict = None) -> Path:
        """Получение пути к файлу кэша."""
        key_data = {'url': url, 'params': params or {}}
        key_str = json.dumps(key_data, sort_keys=True)
        key_hash = hashlib.md5(key_str.encode()).hexdigest()
        return self.cache_dir / f"{key_hash}.json"
    
    def get(self, url: str, params: dict = None) -> dict:
        """Получение данных из кэша."""
        cache_path = self._get_cache_path(url, params)
        
        if not cache_path.exists():
            return None
        
        with open(cache_path, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        # Проверка TTL
        cached_time = datetime.fromisoformat(data['_cached_at'])
        if (datetime.now() - cached_time).total_seconds() > self.ttl_seconds:
            cache_path.unlink()
            return None
        
        return data['response']
    
    def set(self, url: str, params: dict, response: dict):
        """Сохранение данных в кэш."""
        cache_path = self._get_cache_path(url, params)
        
        data = {
            '_cached_at': datetime.now().isoformat(),
            'url': url,
            'params': params,
            'response': response
        }
        
        with open(cache_path, 'w', encoding='utf-8') as f:
            json.dump(data, f, indent=2, ensure_ascii=False)
    
    def clear(self):
        """Очистка кэша."""
        for file in self.cache_dir.glob("*.json"):
            file.unlink()


class CachedAPIClient:
    """API клиент с кэшированием."""
    
    def __init__(self, cache_ttl: int = 300):
        self.cache = FileCache(ttl_seconds=cache_ttl)
        self.session = requests.Session()
    
    def get(self, url: str, params: dict = None, use_cache: bool = True) -> dict:
        """GET запрос с кэшированием."""
        if use_cache:
            cached = self.cache.get(url, params)
            if cached:
                return cached
        
        response = self.session.get(url, params=params, timeout=10)
        response.raise_for_status()
        data = response.json()
        
        if use_cache:
            self.cache.set(url, params, data)
        
        return data
    
    def invalidate(self, url: str, params: dict = None):
        """Инвалидация кэша для конкретного запроса."""
        cache_path = self.cache._get_cache_path(url, params)
        if cache_path.exists():
            cache_path.unlink()
    
    def clear_cache(self):
        """Полная очистка кэша."""
        self.cache.clear()


# Использование
client = CachedAPIClient(cache_ttl=3600)
# data = client.get("https://api.example.com/users", {"page": 1})
4. Оптимизация работы с памятью
4.1. Генераторы вместо списков
python
import sys
import time


def memory_comparison():
    """Сравнение использования памяти."""
    
    # Список (занимает много памяти)
    start_mem = sys.getsizeof([])
    list_data = [i ** 2 for i in range(10_000_000)]
    list_mem = sys.getsizeof(list_data)
    
    # Генератор (экономит память)
    gen_data = (i ** 2 for i in range(10_000_000))
    gen_mem = sys.getsizeof(gen_data)
    
    print(f"Память списка: {list_mem / 1024 / 1024:.2f} MB")
    print(f"Память генератора: {gen_mem} байт")


def process_large_file(filepath: str):
    """
    Обработка большого файла построчно (экономия памяти).
    """
    with open(filepath, 'r') as f:
        for line in f:  # Генератор строк
            yield line.strip()


def chunked_processing(data: list, chunk_size: int = 1000):
    """Обработка данных чанками."""
    for i in range(0, len(data), chunk_size):
        chunk = data[i:i + chunk_size]
        yield chunk
4.2. Использование slots
python
class WithoutSlots:
    """Класс без __slots__ (больше памяти)."""
    def __init__(self, x, y):
        self.x = x
        self.y = y


class WithSlots:
    """Класс с __slots__ (экономия памяти)."""
    __slots__ = ('x', 'y')
    
    def __init__(self, x, y):
        self.x = x
        self.y = y


def compare_memory():
    """Сравнение использования памяти с slots и без."""
    import tracemalloc
    
    tracemalloc.start()
    objects_without = [WithoutSlots(i, i) for i in range(100000)]
    mem_without = tracemalloc.get_traced_memory()[1]
    tracemalloc.stop()
    
    tracemalloc.start()
    objects_with = [WithSlots(i, i) for i in range(100000)]
    mem_with = tracemalloc.get_traced_memory()[1]
    tracemalloc.stop()
    
    print(f"Без __slots__: {mem_without / 1024 / 1024:.2f} MB")
    print(f"С __slots__: {mem_with / 1024 / 1024:.2f} MB")
4.3. Эффективное использование списков
python
class ListOptimization:
    """Оптимизация работы со списками."""
    
    @staticmethod
    def bad_list_creation(n: int) -> list:
        """Плохое создание списка (медленное)."""
        result = []
        for i in range(n):
            result.append(i ** 2)
        return result
    
    @staticmethod
    def good_list_creation(n: int) -> list:
        """Хорошее создание списка (быстрое)."""
        return [i ** 2 for i in range(n)]
    
    @staticmethod
    def preallocated_list(n: int) -> list:
        """С предварительным выделением памяти."""
        result = [0] * n
        for i in range(n):
            result[i] = i ** 2
        return result
    
    @staticmethod
    def compare(n: int = 10_000_000):
        """Сравнение способов создания списков."""
        import time
        
        start = time.perf_counter()
        ListOptimization.bad_list_creation(n)
        bad_time = time.perf_counter() - start
        
        start = time.perf_counter()
        ListOptimization.good_list_creation(n)
        good_time = time.perf_counter() - start
        
        start = time.perf_counter()
        ListOptimization.preallocated_list(n)
        prealloc_time = time.perf_counter() - start
        
        print(f"Медленное создание: {bad_time:.3f} сек")
        print(f"List comprehension: {good_time:.3f} сек")
        print(f"С предвыделением: {prealloc_time:.3f} сек")
5. Практические примеры
5.1. Пример: Оптимизация обработки данных
python
import time
from collections import defaultdict
from functools import lru_cache


class DataProcessor:
    """Класс для обработки данных с оптимизациями."""
    
    @staticmethod
    def bad_grouping(data: list) -> dict:
        """Плохая группировка данных."""
        result = {}
        for item in data:
            category = item['category']
            if category not in result:
                result[category] = []
            result[category].append(item)
        return result
    
    @staticmethod
    def good_grouping(data: list) -> dict:
        """Хорошая группировка (с использованием defaultdict)."""
        result = defaultdict(list)
        for item in data:
            result[item['category']].append(item)
        return dict(result)
    
    @staticmethod
    @lru_cache(maxsize=128)
    def expensive_calculation(x: int, y: int) -> int:
        """Кэшируемое вычисление."""
        time.sleep(0.001)  # Имитация работы
        return x ** y
    
    @staticmethod
    def process_batch(data: list, chunk_size: int = 1000):
        """Обработка данных чанками."""
        results = []
        for i in range(0, len(data), chunk_size):
            chunk = data[i:i + chunk_size]
            # Обработка чанка
            processed = [item * 2 for item in chunk]
            results.extend(processed)
        return results


def optimize_data_processing():
    """Демонстрация оптимизации обработки данных."""
    
    # Тестовые данные
    data = [{'category': f'cat_{i % 10}', 'value': i} for i in range(100000)]
    
    # Сравнение группировки
    start = time.perf_counter()
    DataProcessor.bad_grouping(data)
    bad_time = time.perf_counter() - start
    
    start = time.perf_counter()
    DataProcessor.good_grouping(data)
    good_time = time.perf_counter() - start
    
    print(f"Группировка (плохая): {bad_time:.3f} сек")
    print(f"Группировка (хорошая): {good_time:.3f} сек")
    print(f"Ускорение: {bad_time / good_time:.2f} раз")
5.2. Пример: Кэширование в ETL-процессе
python
from functools import lru_cache
import hashlib
import json


class ETLWithCache:
    """ETL-процесс с кэшированием результатов."""
    
    def __init__(self, cache_size: int = 1000):
        self.cache_size = cache_size
        self._setup_cache()
    
    def _setup_cache(self):
        """Настройка кэша для разных этапов."""
        
        @lru_cache(maxsize=self.cache_size)
        def cached_extract(source_id: str) -> dict:
            """Кэшированное извлечение данных."""
            return self._extract_impl(source_id)
        
        @lru_cache(maxsize=self.cache_size)
        def cached_transform(raw_data_hash: str) -> dict:
            """Кэшированная трансформация."""
            return self._transform_impl(raw_data_hash)
        
        self.cached_extract = cached_extract
        self.cached_transform = cached_transform
    
    def _extract_impl(self, source_id: str) -> dict:
        """Реальная реализация извлечения."""
        # Имитация запроса к БД
        time.sleep(0.1)
        return {"id": source_id, "data": f"Data from {source_id}"}
    
    def _transform_impl(self, raw_data_hash: str) -> dict:
        """Реальная реализация трансформации."""
        # Имитация преобразования
        time.sleep(0.05)
        return {"transformed": raw_data_hash, "processed": True}
    
    def process(self, source_id: str) -> dict:
        """Обработка данных с кэшированием."""
        # Извлечение (с кэшем)
        raw_data = self.cached_extract(source_id)
        
        # Создание хеша для кэша трансформации
        data_hash = hashlib.md5(
            json.dumps(raw_data, sort_keys=True).encode()
        ).hexdigest()
        
        # Трансформация (с кэшем)
        transformed = self.cached_transform(data_hash)
        
        return transformed
    
    def get_cache_info(self):
        """Информация о кэше."""
        return {
            'extract': self.cached_extract.cache_info(),
            'transform': self.cached_transform.cache_info()
        }
    
    def clear_cache(self):
        """Очистка кэша."""
        self.cached_extract.cache_clear()
        self.cached_transform.cache_clear()
6. Контрольные вопросы
Как выявить узкие места в коде без использования профайлера?

В чём разница между преждевременной и осознанной оптимизацией?

Как работает functools.lru_cache и когда его стоит использовать?

Какие структуры данных лучше использовать для частого поиска элементов?

Почему конкатенация строк через + в цикле медленная?

Когда стоит использовать генераторы вместо списков?

Что такое __slots__ и как он экономит память?

Как кэширование может ускорить ETL-процесс?

Какие операции являются самыми дорогими в Python?

Как измерить улучшение производительности после оптимизации?

7. Практическое задание
Задание 1 (базовое)
Напишите функцию, которая находит все простые числа до N. Реализуйте три версии (наивная, с решетом Эратосфена, с оптимизациями). Сравните их производительность.

Задание 2 (среднее)
Реализуйте декоратор @cached(ttl=60), который кэширует результаты функции на 60 секунд.

Задание 3 (сложное)
Оптимизируйте ETL-скрипт обработки 1 млн записей:

Добавьте кэширование результатов

Оптимизируйте циклы

Используйте генераторы для экономии памяти

Сравните производительность до и после

8. Шпаргалка
python
# === КЭШИРОВАНИЕ ===
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_func(x):
    return x ** 2

# === ГЕНЕРАТОРЫ ===
gen = (i ** 2 for i in range(1000000))
for val in gen:
    pass

# === LIST COMPREHENSION ===
squares = [i ** 2 for i in range(1000000)]

# === DEFAULTDICT ===
from collections import defaultdict
d = defaultdict(list)

# === СТРОКИ ===
result = " ".join(words)  # вместо конкатенации в цикле

# === ПАМЯТЬ ===
__slots__ = ('x', 'y')  # экономия памяти в классах

# === ОПТИМИЗАЦИЯ ЦИКЛОВ ===
# Плохо: for i in range(len(data)):
# Хорошо: for item in data:

# === ПРЕДВЫДЕЛЕНИЕ ===
arr = [0] * n  # выделение памяти заранее
Итог лекции
Вы сегодня:

✅ Научились выявлять узкие места в коде

✅ Изучили стратегии оптимизации (от архитектурных до микрооптимизаций)

✅ Освоили кэширование с lru_cache и создание собственных кэшей

✅ Оптимизировали работу с памятью (генераторы, slots, предвыделение)

✅ Применили техники оптимизации в реальных примерах

Теперь ваш код будет работать быстрее и эффективнее!
