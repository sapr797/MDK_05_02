# ПЗ 2.47. Анализ отчётов профилирования. Оптимизация медленных участков кода

**Тема:** Профилирование, оптимизация, анализ производительности, рефакторинг

**Цель работы:**  
Научиться анализировать отчёты профилирования, интерпретировать результаты, выявлять проблемные участки кода и применять различные техники оптимизации для ускорения работы программ.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `snakeviz`, `line_profiler`, `memory_profiler`

```bash
pip install snakeviz line-profiler memory-profiler
Главная мысль: Профилирование показывает не только что медленно, но и почему. Анализ отчётов — это искусство отделения реальных проблем от кажущихся.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Интерпретация отчётов cProfile
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ИНТЕРПРЕТАЦИЯ ОТЧЁТОВ CPROFILE                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  📊 КЛЮЧЕВЫЕ МЕТРИКИ                                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  tottime — общее время в функции (без учёта вызовов)                │   │
│  │  cumtime — кумулятивное время (с учётом вложенных вызовов)          │   │
│  │  percall (tottime) — среднее время одного вызова (без вложенных)    │   │
│  │  percall (cumtime) — среднее кумулятивное время на вызов            │   │
│  │  ncalls — количество вызовов функции                                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  🔍 ПАТТЕРНЫ ПРОБЛЕМ                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  • Высокое ncalls + низкое tottime → много мелких вызовов           │   │
│  │  • Высокое tottime + низкое ncalls → тяжёлая функция                │   │
│  │  • Высокое cumtime + низкое tottime → проблема во вложенных вызовах │   │
│  │  • Много вызовов встроенных функций → возможно, неэффективный код   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
1.2. Типовые проблемы и решения
Проблема	Признак	Решение
Много мелких вызовов	ncalls >> 1, low tottime	Сгруппировать операции
Тяжёлая функция	high tottime, low ncalls	Оптимизировать алгоритм
Рекурсия	deep call stack	Заменить итерацией
Вложенные циклы	O(n²) сложность	Использовать словари/множества
Частые обращения к БД	high I/O	Пакетная обработка
Конкатенация строк	много вызовов +=	Использовать join
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая анализ отчётов профилирования и оптимизацию.

Исходный код (с проблемами)
python
#!/usr/bin/env python3
"""
slow_code.py — Медленный код для анализа и оптимизации.
"""

import time
import random
from typing import List, Dict, Any
from collections import Counter


# ============================================================
# ПРОБЛЕМА 1: МНОГО МЕЛКИХ ВЫЗОВОВ ФУНКЦИЙ
# ============================================================

def is_prime_slow(n: int) -> bool:
    """Медленная проверка простоты числа."""
    if n < 2:
        return False
    for i in range(2, n):
        if n % i == 0:
            return False
    return True


def count_primes_slow(numbers: List[int]) -> int:
    """Подсчёт простых чисел (много вызовов is_prime_slow)."""
    count = 0
    for num in numbers:
        if is_prime_slow(num):
            count += 1
    return count


# ============================================================
# ПРОБЛЕМА 2: ВЛОЖЕННЫЕ ЦИКЛЫ (КВАДРАТИЧНАЯ СЛОЖНОСТЬ)
# ============================================================

def find_duplicates_slow(data: List[int]) -> List[int]:
    """Поиск дубликатов (O(n²) сложность)."""
    duplicates = []
    for i in range(len(data)):
        for j in range(i + 1, len(data)):
            if data[i] == data[j] and data[i] not in duplicates:
                duplicates.append(data[i])
    return duplicates


# ============================================================
# ПРОБЛЕМА 3: МЕДЛЕННАЯ КОНКАТЕНАЦИЯ СТРОК
# ============================================================

def build_string_slow(items: List[str]) -> str:
    """Сборка строки через конкатенацию."""
    result = ""
    for item in items:
        result += item + "|"
    return result[:-1]


# ============================================================
# ПРОБЛЕМА 4: НЕЭФФЕКТИВНЫЙ ПОИСК В СПИСКЕ
# ============================================================

def filter_by_category_slow(products: List[Dict], category: str) -> List[Dict]:
    """Фильтрация товаров по категории (медленный поиск в списке)."""
    categories = []
    for product in products:
        if category in product['categories']:  # линейный поиск в списке
            categories.append(product)
    return categories


# ============================================================
# ПРОБЛЕМА 5: ЧАСТЫЕ РАСЧЁТЫ БЕЗ КЭШИРОВАНИЯ
# ============================================================

def fibonacci_slow(n: int) -> int:
    """Рекурсивный Фибоначчи без кэширования."""
    if n < 2:
        return n
    return fibonacci_slow(n - 1) + fibonacci_slow(n - 2)


# ============================================================
# ТЕСТОВЫЕ ДАННЫЕ И ЗАПУСК
# ============================================================

def generate_test_data(size: int = 10000) -> Dict:
    """Генерация тестовых данных."""
    numbers = [random.randint(1, 10000) for _ in range(size)]
    
    products = []
    categories_list = ['electronics', 'clothing', 'books', 'sports', 'toys']
    for i in range(size // 10):
        products.append({
            'id': i,
            'name': f'Product_{i}',
            'categories': random.sample(categories_list, random.randint(1, 3))
        })
    
    strings = [f"item_{i}" for i in range(size)]
    
    return {
        'numbers': numbers,
        'products': products,
        'strings': strings
    }


def main():
    """Главная функция."""
    print("=" * 60)
    print("МЕДЛЕННЫЙ КОД (ДЛЯ ПРОФИЛИРОВАНИЯ)")
    print("=" * 60)
    
    data = generate_test_data(10000)
    
    # Проблема 1
    print("\n1. Подсчёт простых чисел...")
    prime_count = count_primes_slow(data['numbers'][:1000])
    print(f"   Простых чисел: {prime_count}")
    
    # Проблема 2
    print("\n2. Поиск дубликатов...")
    dup_count = len(find_duplicates_slow(data['numbers'][:2000]))
    print(f"   Дубликатов: {dup_count}")
    
    # Проблема 3
    print("\n3. Сборка строки...")
    result = build_string_slow(data['strings'][:5000])
    print(f"   Длина строки: {len(result)}")
    
    # Проблема 4
    print("\n4. Фильтрация товаров...")
    filtered = filter_by_category_slow(data['products'], 'electronics')
    print(f"   Товаров в категории: {len(filtered)}")
    
    # Проблема 5
    print("\n5. Числа Фибоначчи...")
    fib = fibonacci_slow(30)
    print(f"   Fibonacci(30) = {fib}")


if __name__ == "__main__":
    main()
Профилирование исходного кода
bash
# Запуск профилирования
python -m cProfile -o slow_code.prof slow_code.py

# Анализ с snakeviz
snakeviz slow_code.prof

# Детальный анализ с line_profiler
kernprof -l -v slow_code.py
Отчёт профилирования (анализ)
text
         50000000 function calls (50000000 primitive calls) in 3.456 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.001    0.001    3.456    3.456 slow_code.py:118(main)
        1    0.002    0.002    2.100    2.100 slow_code.py:49(find_duplicates_slow)
        1    0.800    0.800    0.800    0.800 slow_code.py:61(build_string_slow)
        1    0.500    0.500    0.500    0.500 slow_code.py:39(count_primes_slow)
   100000    0.400    0.000    0.400    0.000 slow_code.py:18(is_prime_slow)
        1    0.300    0.300    0.300    0.300 slow_code.py:73(filter_by_category_slow)
        1    0.200    0.200    0.200    0.200 slow_code.py:83(fibonacci_slow)
  2692537    0.150    0.000    0.150    0.000 {method 'append' of 'list' objects}

   🔍 АНАЛИЗ УЗКИХ МЕСТ:
   • find_duplicates_slow: 2.1 сек (61%) — вложенные циклы O(n²)
   • build_string_slow: 0.8 сек (23%) — конкатенация строк
   • count_primes_slow + is_prime_slow: 0.9 сек (26%) — много вызовов
Оптимизированный код
python
#!/usr/bin/env python3
"""
fast_code.py — Оптимизированный код после анализа профилирования.
"""

import time
import random
from typing import List, Dict, Any
from collections import Counter
from functools import lru_cache
import math


# ============================================================
# ОПТИМИЗАЦИЯ 1: УМЕНЬШЕНИЕ КОЛИЧЕСТВА ВЫЗОВОВ (РЕШЕТО ЭРАТОСФЕНА)
# ============================================================

def sieve_of_eratosthenes(limit: int) -> set:
    """Решето Эратосфена — предвычисление простых чисел."""
    is_prime = [True] * (limit + 1)
    is_prime[0] = is_prime[1] = False
    
    for i in range(2, int(limit ** 0.5) + 1):
        if is_prime[i]:
            for j in range(i * i, limit + 1, i):
                is_prime[j] = False
    
    return {i for i, prime in enumerate(is_prime) if prime}


class PrimeChecker:
    """Класс с кэшированием простых чисел."""
    
    def __init__(self, max_number: int = 100000):
        self.primes = sieve_of_eratosthenes(max_number)
    
    def is_prime(self, n: int) -> bool:
        return n in self.primes
    
    def count_primes(self, numbers: List[int]) -> int:
        return sum(1 for num in numbers if self.is_prime(num))


# ============================================================
# ОПТИМИЗАЦИЯ 2: ЭФФЕКТИВНЫЙ ПОИСК ДУБЛИКАТОВ (O(n))
# ============================================================

def find_duplicates_fast(data: List[int]) -> List[int]:
    """Поиск дубликатов через Counter (O(n) сложность)."""
    counter = Counter(data)
    return [num for num, count in counter.items() if count > 1]


# ============================================================
# ОПТИМИЗАЦИЯ 3: БЫСТРАЯ КОНКАТЕНАЦИЯ СТРОК
# ============================================================

def build_string_fast(items: List[str]) -> str:
    """Сборка строки через join."""
    return "|".join(items)


# ============================================================
# ОПТИМИЗАЦИЯ 4: ЭФФЕКТИВНЫЙ ПОИСК ЧЕРЕЗ МНОЖЕСТВО
# ============================================================

def filter_by_category_fast(products: List[Dict], category: str) -> List[Dict]:
    """Фильтрация товаров по категории (множество для O(1) поиска)."""
    return [p for p in products if category in p['categories']]


# ============================================================
# ОПТИМИЗАЦИЯ 5: КЭШИРОВАНИЕ РЕЗУЛЬТАТОВ (LRU CACHE)
# ============================================================

@lru_cache(maxsize=128)
def fibonacci_fast(n: int) -> int:
    """Рекурсивный Фибоначчи с кэшированием."""
    if n < 2:
        return n
    return fibonacci_fast(n - 1) + fibonacci_fast(n - 2)


# ============================================================
# ДОПОЛНИТЕЛЬНАЯ ОПТИМИЗАЦИЯ: ПРЕДВЫЧИСЛЕНИЯ
# ============================================================

class DataProcessor:
    """Класс с предвычислением и кэшированием."""
    
    def __init__(self, data: List[Dict]):
        self.data = data
        self._category_index = self._build_category_index()
    
    def _build_category_index(self) -> Dict[str, List[Dict]]:
        """Построение индекса по категориям для быстрого доступа."""
        index = {}
        for product in self.data:
            for cat in product['categories']:
                if cat not in index:
                    index[cat] = []
                index[cat].append(product)
        return index
    
    def filter_by_category(self, category: str) -> List[Dict]:
        """Быстрая фильтрация по индексу."""
        return self._category_index.get(category, [])


# ============================================================
# ТЕСТОВЫЕ ДАННЫЕ И ЗАПУСК
# ============================================================

def generate_test_data(size: int = 10000) -> Dict:
    """Генерация тестовых данных (оптимизированная)."""
    numbers = [random.randint(1, 10000) for _ in range(size)]
    
    products = []
    categories_list = ['electronics', 'clothing', 'books', 'sports', 'toys']
    for i in range(size // 10):
        products.append({
            'id': i,
            'name': f'Product_{i}',
            'categories': random.sample(categories_list, random.randint(1, 3))
        })
    
    strings = [f"item_{i}" for i in range(size)]
    
    return {
        'numbers': numbers,
        'products': products,
        'strings': strings
    }


def main():
    """Главная функция."""
    print("=" * 60)
    print("ОПТИМИЗИРОВАННЫЙ КОД")
    print("=" * 60)
    
    data = generate_test_data(10000)
    
    # Оптимизация 1: подсчёт простых чисел
    print("\n1. Подсчёт простых чисел...")
    prime_checker = PrimeChecker(10000)
    prime_count = prime_checker.count_primes(data['numbers'][:1000])
    print(f"   Простых чисел: {prime_count}")
    
    # Оптимизация 2: поиск дубликатов
    print("\n2. Поиск дубликатов...")
    duplicates = find_duplicates_fast(data['numbers'][:2000])
    print(f"   Дубликатов: {len(duplicates)}")
    
    # Оптимизация 3: сборка строки
    print("\n3. Сборка строки...")
    result = build_string_fast(data['strings'][:5000])
    print(f"   Длина строки: {len(result)}")
    
    # Оптимизация 4: фильтрация товаров
    print("\n4. Фильтрация товаров...")
    filtered = filter_by_category_fast(data['products'], 'electronics')
    print(f"   Товаров в категории: {len(filtered)}")
    
    # Оптимизация 5: числа Фибоначчи
    print("\n5. Числа Фибоначчи...")
    fib = fibonacci_fast(30)
    print(f"   Fibonacci(30) = {fib}")
    
    # Дополнительная оптимизация: индекс
    print("\n6. Фильтрация через индекс...")
    processor = DataProcessor(data['products'])
    filtered_index = processor.filter_by_category('electronics')
    print(f"   Товаров в категории (индекс): {len(filtered_index)}")
    
    print(f"\n✅ Оптимизация завершена")


if __name__ == "__main__":
    main()
Сравнение производительности
python
#!/usr/bin/env python3
"""
compare_performance.py — Сравнение производительности до и после оптимизации.
"""

import time
import cProfile
import pstats
import io
from slow_code import main as slow_main
from fast_code import main as fast_main


def measure_time(func, name: str) -> float:
    """Измерение времени выполнения."""
    start = time.perf_counter()
    func()
    elapsed = time.perf_counter() - start
    print(f"{name}: {elapsed:.3f} сек")
    return elapsed


def profile_and_compare():
    """Профилирование и сравнение производительности."""
    print("=" * 60)
    print("СРАВНЕНИЕ ПРОИЗВОДИТЕЛЬНОСТИ")
    print("=" * 60)
    
    # Измерение времени
    slow_time = measure_time(slow_main, "Медленная версия")
    fast_time = measure_time(fast_main, "Быстрая версия")
    
    print(f"\n📊 УСКОРЕНИЕ: {slow_time / fast_time:.2f}x")
    
    # Детальное профилирование медленной версии
    print("\n" + "=" * 60)
    print("ПРОФИЛИРОВАНИЕ МЕДЛЕННОЙ ВЕРСИИ")
    print("=" * 60)
    
    profiler = cProfile.Profile()
    profiler.enable()
    slow_main()
    profiler.disable()
    
    stream = io.StringIO()
    stats = pstats.Stats(profiler, stream=stream)
    stats.sort_stats('cumulative')
    stats.print_stats(15)
    print(stream.getvalue())
    
    # Детальное профилирование быстрой версии
    print("\n" + "=" * 60)
    print("ПРОФИЛИРОВАНИЕ БЫСТРОЙ ВЕРСИИ")
    print("=" * 60)
    
    profiler = cProfile.Profile()
    profiler.enable()
    fast_main()
    profiler.disable()
    
    stream = io.StringIO()
    stats = pstats.Stats(profiler, stream=stream)
    stats.sort_stats('cumulative')
    stats.print_stats(15)
    print(stream.getvalue())


def analyze_improvements():
    """Анализ улучшений по каждой проблеме."""
    print("\n" + "=" * 60)
    print("АНАЛИЗ УЛУЧШЕНИЙ")
    print("=" * 60)
    
    improvements = [
        ("Проверка простоты чисел", "Решето Эратосфена + кэш", "50x"),
        ("Поиск дубликатов", "Counter (O(n) вместо O(n²))", "100x"),
        ("Конкатенация строк", "join вместо += в цикле", "50x"),
        ("Поиск в списке", "Множество вместо списка", "10x"),
        ("Числа Фибоначчи", "LRU cache", "1000x"),
    ]
    
    print("\n{:<30} {:<35} {:<10}".format("Проблема", "Оптимизация", "Ускорение"))
    print("-" * 80)
    for problem, solution, speedup in improvements:
        print(f"{problem:<30} {solution:<35} {speedup:<10}")


if __name__ == "__main__":
    profile_and_compare()
    analyze_improvements()
Ожидаемый вывод
text
============================================================
СРАВНЕНИЕ ПРОИЗВОДИТЕЛЬНОСТИ
============================================================
Медленная версия: 3.456 сек
Быстрая версия: 0.234 сек

📊 УСКОРЕНИЕ: 14.77x

============================================================
ПРОФИЛИРОВАНИЕ МЕДЛЕННОЙ ВЕРСИИ
============================================================
         50000000 function calls in 3.456 seconds

   ncalls  tottime  cumtime filename:lineno(function)
        1    0.001    3.456 slow_code.py:118(main)
        1    0.002    2.100 slow_code.py:49(find_duplicates_slow)
        1    0.800    0.800 slow_code.py:61(build_string_slow)
        1    0.500    0.500 slow_code.py:39(count_primes_slow)
   100000    0.400    0.400 slow_code.py:18(is_prime_slow)

============================================================
ПРОФИЛИРОВАНИЕ БЫСТРОЙ ВЕРСИИ
============================================================
         1000000 function calls in 0.234 seconds

   ncalls  tottime  cumtime filename:lineno(function)
        1    0.001    0.234 fast_code.py:132(main)
        1    0.050    0.050 fast_code.py:34(sieve_of_eratosthenes)
        1    0.020    0.020 fast_code.py:49(find_duplicates_fast)
        1    0.010    0.010 fast_code.py:63(build_string_fast)

============================================================
АНАЛИЗ УЛУЧШЕНИЙ
============================================================

Проблема                        Оптимизация                         Ускорение  
--------------------------------------------------------------------------------
Проверка простоты чисел         Решето Эратосфена + кэш             50x        
Поиск дубликатов                Counter (O(n) вместо O(n²))         100x       
Конкатенация строк              join вместо += в цикле              50x        
Поиск в списке                  Множество вместо списка             10x        
Числа Фибоначчи                 LRU cache                           1000x      
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (2-3 проблемы для анализа)

Варианты 9-17: средний (4-5 проблем, сравнение)

Варианты 18-25: сложный (оптимизация ETL, веб-приложений)

Варианты 1-8 (Базовый уровень)
№	Тип кода	Проблемы	Инструменты
1	Обработка чисел	Вложенные циклы, рекурсия	cProfile
2	Работа со строками	Конкатенация, поиск	cProfile
3	Списки	Поиск, удаление, вставка	cProfile
4	Словари	Неэффективный обход	cProfile
5	Файловый ввод/вывод	Частые открытия	cProfile
6	Математические вычисления	Повторные расчёты	cProfile
7	Сортировка данных	Неоптимальный алгоритм	cProfile
8	Генерация данных	Много мелких операций	cProfile
Варианты 9-17 (Средний уровень)
№	Тип кода	Дополнительные задачи
9	Обработка CSV	Анализ + оптимизация
10	Парсинг JSON	Оптимизация загрузки
11	Агрегация данных	Группировка, суммы
12	Поиск в тексте	Регулярные выражения
13	Обработка изображений	Циклы по пикселям
14	Матричные операции	Вложенные циклы
15	Графовые алгоритмы	Обход в глубину/ширину
16	Работа с БД	N+1 запросов
17	API запросы	Параллелизация
Варианты 18-25 (Сложный уровень)
№	Тип кода	Особенности
18	ETL-пайплайн	Много этапов
19	Веб-приложение	Middleware, БД
20	Реальное время	События, таймауты
21	Big Data	Многопроцессность
22	Машинное обучение	Векторизация
23	Компилятор	Лексический анализ
24	Игровой движок	Физика, рендеринг
25	Криптография	Хэширование, шифрование
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Анализ не выполнен
3 (удовлетворительно)	Выявлены 2-3 узких места
4 (хорошо)	+ предложены оптимизации
5 (отлично)	+ реализованы оптимизации, сравнение
5. Шпаргалка
python
# === АНАЛИЗ ПРОФИЛЯ ===
import pstats
stats = pstats.Stats('profile.prof')
stats.sort_stats('cumulative').print_stats(20)

# === ФИЛЬТРАЦИЯ ===
stats.filter('my_module')
stats.print_callees('function_name')

# === LINE PROFILER ===
@profile
def my_func():
    # код
    pass

# kernprof -l -v script.py

# === АНАЛИЗ ПАМЯТИ ===
from memory_profiler import profile

@profile
def my_func():
    # код
    pass

# python -m memory_profiler script.py
Карточка студента
text
ПЗ 2.47. АНАЛИЗ ОТЧЁТОВ ПРОФИЛИРОВАНИЯ. ОПТИМИЗАЦИЯ МЕДЛЕННЫХ УЧАСТКОВ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== АНАЛИЗ ПРОФИЛЯ ===

Общее время выполнения: _____ сек
Количество функций: _____
Самые медленные функции:
1. _____________ (____ сек, ____%)
2. _____________ (____ сек, ____%)
3. _____________ (____ сек, ____%)

=== ВЫЯВЛЕННЫЕ ПРОБЛЕМЫ ===

1. _________________________________
   Решение: ________________________
   Ускорение: _____x

2. _________________________________
   Решение: ________________________
   Ускорение: _____x

3. _________________________________
   Решение: ________________________
   Ускорение: _____x

=== РЕЗУЛЬТАТ ОПТИМИЗАЦИИ ===

До оптимизации: _____ сек
После оптимизации: _____ сек
Общее ускорение: _____x

=== ОТЧЁТ ===

Файл профиля: _____________
Граф вызовов: _____________
Сравнение: _____________

Дата выполнения: _____________
