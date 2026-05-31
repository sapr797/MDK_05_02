# Тема 2.37. Профилирование кода. cProfile

**Цель лекции:**  
Изучить инструменты профилирования Python-кода, освоить работу с модулем cProfile, научиться анализировать результаты профилирования, находить узкие места и оптимизировать производительность.

> Главная мысль: **Не оптимизируйте вслепую. Сначала измерьте, где находится реальное узкое место. Профилирование — это компас в мире оптимизации.**

---

## Содержание

1. [Введение в профилирование](#1-введение-в-профилирование)
2. [Модуль cProfile](#2-модуль-cprofile)
3. [Анализ результатов профилирования](#3-анализ-результатов-профилирования)
4. [Визуализация результатов](#4-визуализация-результатов)
5. [Сравнение производительности](#5-сравнение-производительности)
6. [Профилирование в реальных проектах](#6-профилирование-в-реальных-проектах)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Введение в профилирование

### 1.1. Что такое профилирование

**Профилирование** — это процесс измерения времени выполнения различных частей программы для выявления узких мест.
┌─────────────────────────────────────────────────────────────────────────────┐
│ ПРОЦЕСС ПРОФИЛИРОВАНИЯ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ ШАГ 1: ЗАПУСК ПРОФИЛИРОВАНИЯ │ │
│ │ python -m cProfile script.py │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │ │
│ ▼ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ ШАГ 2: СБОР ДАННЫХ │ │
│ │ • Количество вызовов каждой функции │ │
│ │ • Общее время выполнения │ │
│ │ • Время на один вызов │ │
│ │ • Время вложенных вызовов │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │ │
│ ▼ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ ШАГ 3: АНАЛИЗ РЕЗУЛЬТАТОВ │ │
│ │ • Найти функции с наибольшим временем │ │
│ │ • Найти функции с наибольшим количеством вызовов │ │
│ │ • Найти узкие места │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │ │
│ ▼ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ ШАГ 4: ОПТИМИЗАЦИЯ │ │
│ │ • Переписать медленные участки │ │
│ │ • Использовать кэширование │ │
│ │ • Изменить алгоритм │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Типы профилирования

| Тип | Описание | Инструменты |
|-----|----------|-------------|
| **Статистическое** | Сэмплирование стека через интервалы | `py-spy`, `vmprof` |
| **Инструментальное** | Добавление кода для измерения | `cProfile`, `profile` |
| **Линейное** | Построчное профилирование | `line_profiler` |
| **Памяти** | Профилирование использования памяти | `memory_profiler` |

---

## 2. Модуль cProfile

### 2.1. Основы cProfile

```python
import cProfile
import pstats
import io


def slow_function():
    """Медленная функция для профилирования."""
    total = 0
    for i in range(1000000):
        total += i ** 2
    return total


def fast_function():
    """Быстрая функция для сравнения."""
    return sum(i ** 2 for i in range(1000000))


def main():
    slow_function()
    fast_function()


# Способ 1: Запуск профилирования из кода
if __name__ == "__main__":
    profiler = cProfile.Profile()
    profiler.enable()
    main()
    profiler.disable()
    
    # Сохранение результатов
    profiler.dump_stats('profile_output.prof')
    
    # Вывод в консоль
    stream = io.StringIO()
    stats = pstats.Stats(profiler, stream=stream)
    stats.sort_stats('cumulative')
    stats.print_stats(10)
    print(stream.getvalue())
2.2. Запуск профилирования из командной строки
bash
# Профилирование скрипта
python -m cProfile my_script.py

# Сохранение результатов в файл
python -m cProfile -o output.prof my_script.py

# Сортировка по времени
python -m cProfile -s cumulative my_script.py

# Ограничение вывода
python -m cProfile my_script.py | head -20
2.3. Расшифровка вывода cProfile
text
         10000005 function calls (10000003 primitive calls) in 1.234 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    1.234    1.234 my_script.py:8(main)
        1    0.567    0.567    0.890    0.890 my_script.py:4(slow_function)
  1000000    0.323    0.000    0.323    0.000 {built-in method builtins.pow}
        1    0.344    0.344    0.344    0.344 my_script.py:12(fast_function)
Колонка	Описание
ncalls	Количество вызовов функции
tottime	Общее время в функции (без вложенных)
percall	Среднее время на один вызов (tottime/ncalls)
cumtime	Кумулятивное время (с вложенными)
percall	Среднее кумулятивное время на вызов
filename:lineno(function)	Информация о функции
3. Анализ результатов профилирования
3.1. Работа с pstats
python
import cProfile
import pstats
from pstats import SortKey


def create_stats(profiler: cProfile.Profile) -> pstats.Stats:
    """Создание объекта статистики."""
    stats = pstats.Stats(profiler)
    return stats


def analyze_profile(profiler: cProfile.Profile):
    """Комплексный анализ профиля."""
    stats = pstats.Stats(profiler)
    
    print("=" * 60)
    print("ТОП-10 ПО КУМУЛЯТИВНОМУ ВРЕМЕНИ")
    print("=" * 60)
    stats.sort_stats(SortKey.CUMULATIVE).print_stats(10)
    
    print("\n" + "=" * 60)
    print("ТОП-10 ПО ОБЩЕМУ ВРЕМЕНИ")
    print("=" * 60)
    stats.sort_stats(SortKey.TIME).print_stats(10)
    
    print("\n" + "=" * 60)
    print("ТОП-10 ПО КОЛИЧЕСТВУ ВЫЗОВОВ")
    print("=" * 60)
    stats.sort_stats(SortKey.CALLS).print_stats(10)
    
    print("\n" + "=" * 60)
    print("ФУНКЦИИ С НАИБОЛЬШИМ КОЛИЧЕСТВОМ ВЫЗОВОВ")
    print("=" * 60)
    stats.sort_stats(SortKey.CALLS).print_stats(10)
    
    return stats


def filter_by_module(stats: pstats.Stats, module: str):
    """Фильтрация по модулю."""
    stats = pstats.Stats(stats)
    stats = stats.filter(module)
    stats.print_stats()
    return stats


def get_callers(stats: pstats.Stats, function: str):
    """Получение вызывающих функций."""
    stats = pstats.Stats(stats)
    stats.print_callers(function)


def get_callees(stats: pstats.Stats, function: str):
    """Получение вызываемых функций."""
    stats = pstats.Stats(stats)
    stats.print_callees(function)
3.2. Поиск узких мест
python
def identify_bottlenecks(stats: pstats.Stats, threshold: float = 1.0):
    """
    Выявление функций, которые занимают больше threshold процентов времени.
    """
    total_time = stats.total_tt
    
    results = []
    for func, (cc, nc, tt, ct, callers) in stats.stats.items():
        percent = (ct / total_time) * 100
        if percent > threshold:
            results.append({
                'function': func,
                'cumulative_time': ct,
                'percent': percent,
                'calls': nc
            })
    
    # Сортировка по проценту времени
    results.sort(key=lambda x: x['percent'], reverse=True)
    
    print(f"\nФункции, занимающие более {threshold}% времени:")
    for r in results[:10]:
        print(f"  {r['function'][2]}: {r['percent']:.1f}% ({r['cumulative_time']:.3f}s, {r['calls']} вызовов)")
    
    return results
4. Визуализация результатов
4.1. Установка графических инструментов
bash
# Установка snakeviz
pip install snakeviz

# Установка gprof2dot и graphviz
pip install gprof2dot
# graphviz: https://graphviz.org/download/
4.2. Визуализация с snakeviz
bash
# Запуск профилирования
python -m cProfile -o output.prof my_script.py

# Визуализация с snakeviz
snakeviz output.prof

# Откроется браузер с интерактивной визуализацией
python
import cProfile
import subprocess
from pathlib import Path


def profile_and_visualize(func, *args, output_file='profile_output.prof'):
    """
    Профилирование функции и запуск визуализации.
    """
    profiler = cProfile.Profile()
    profiler.enable()
    result = func(*args)
    profiler.disable()
    
    profiler.dump_stats(output_file)
    
    # Запуск snakeviz
    subprocess.run(['snakeviz', output_file])
    
    return result
4.3. Генерация графа вызовов
python
import subprocess
from pathlib import Path


def generate_call_graph(prof_file: str, output_file: str = 'call_graph.png'):
    """
    Генерация графа вызовов с помощью gprof2dot и graphviz.
    """
    cmd = f'gprof2dot -f pstats {prof_file} | dot -Tpng -o {output_file}'
    subprocess.run(cmd, shell=True)
    print(f"Граф вызовов сохранён в {output_file}")
    return output_file
4.4. Интерактивный анализ
python
import cProfile
import pstats
from IPython.display import display, HTML


class InteractiveProfiler:
    """Интерактивный профилировщик для Jupyter Notebook."""
    
    def __init__(self):
        self.profiler = None
        self.stats = None
    
    def start(self):
        self.profiler = cProfile.Profile()
        self.profiler.enable()
    
    def stop(self):
        self.profiler.disable()
        self.stats = pstats.Stats(self.profiler)
    
    def show_top(self, n: int = 20):
        stream = io.StringIO()
        self.stats.sort_stats(SortKey.CUMULATIVE).print_stats(n)
        display(HTML(f"<pre>{stream.getvalue()}</pre>"))
    
    def search(self, pattern: str):
        stream = io.StringIO()
        self.stats.sort_stats(SortKey.CUMULATIVE).print_stats()
        filtered = [line for line in stream.getvalue().split('\n') if pattern in line]
        display(HTML(f"<pre>{chr(10).join(filtered[:30])}</pre>"))
    
    def save(self, filename: str):
        self.profiler.dump_stats(filename)
        print(f"Профиль сохранён в {filename}")
5. Сравнение производительности
5.1. A/B тестирование производительности
python
import time
import cProfile
import io
import pstats


class PerformanceComparison:
    """Сравнение производительности двух реализаций."""
    
    def __init__(self, name_a: str, func_a, name_b: str, func_b):
        self.name_a = name_a
        self.func_a = func_a
        self.name_b = name_b
        self.func_b = func_b
        self.results_a = {}
        self.results_b = {}
    
    def profile_function(self, func, name: str, *args, **kwargs):
        """Профилирование отдельной функции."""
        profiler = cProfile.Profile()
        profiler.enable()
        result = func(*args, **kwargs)
        profiler.disable()
        
        stream = io.StringIO()
        stats = pstats.Stats(profiler, stream=stream)
        stats.sort_stats('cumulative')
        stats.print_stats(10)
        
        return {
            'result': result,
            'stats': stats,
            'output': stream.getvalue(),
            'total_time': stats.total_tt
        }
    
    def run(self, *args, **kwargs):
        """Запуск сравнения."""
        print("=" * 60)
        print("СРАВНЕНИЕ ПРОИЗВОДИТЕЛЬНОСТИ")
        print(f"Реализация A: {self.name_a}")
        print(f"Реализация B: {self.name_b}")
        print("=" * 60)
        
        self.results_a = self.profile_function(self.func_a, self.name_a, *args, **kwargs)
        self.results_b = self.profile_function(self.func_b, self.name_b, *args, **kwargs)
        
        self._print_comparison()
        return self.results_a['result'] if self.results_a['total_time'] < self.results_b['total_time'] else self.results_b['result']
    
    def _print_comparison(self):
        """Вывод сравнения."""
        time_a = self.results_a['total_time']
        time_b = self.results_b['total_time']
        
        print("\nРЕЗУЛЬТАТЫ:")
        print(f"  {self.name_a}: {time_a:.4f} сек")
        print(f"  {self.name_b}: {time_b:.4f} сек")
        
        if time_a < time_b:
            speedup = time_b / time_a
            print(f"\n  ✅ {self.name_a} быстрее в {speedup:.2f} раз")
        else:
            speedup = time_a / time_b
            print(f"\n  ✅ {self.name_b} быстрее в {speedup:.2f} раз")
        
        print(f"\nДЕТАЛИ ПРОФИЛИРОВАНИЯ {self.name_a}:")
        print(self.results_a['output'])
        
        print(f"\nДЕТАЛИ ПРОФИЛИРОВАНИЯ {self.name_b}:")
        print(self.results_b['output'])
5.2. Пример сравнения алгоритмов
python
# Реализация A: классическая
def sum_squares_a(n: int) -> int:
    total = 0
    for i in range(n):
        total += i ** 2
    return total

# Реализация B: с использованием генератора
def sum_squares_b(n: int) -> int:
    return sum(i ** 2 for i in range(n))

# Реализация C: с использованием map
def sum_squares_c(n: int) -> int:
    return sum(map(lambda x: x ** 2, range(n)))

# Реализация D: формула суммы квадратов
def sum_squares_d(n: int) -> int:
    return n * (n - 1) * (2 * n - 1) // 6


# Сравнение
comparison = PerformanceComparison(
    "Loop", sum_squares_a,
    "Generator", sum_squares_b
)
comparison.run(1000000)
6. Профилирование в реальных проектах
6.1. Декоратор для профилирования
python
import functools
import cProfile
import io
import pstats
from typing import Callable, Any


def profile(func: Callable) -> Callable:
    """
    Декоратор для профилирования функции.
    """
    @functools.wraps(func)
    def wrapper(*args, **kwargs) -> Any:
        profiler = cProfile.Profile()
        profiler.enable()
        result = func(*args, **kwargs)
        profiler.disable()
        
        # Анализ результатов
        stream = io.StringIO()
        stats = pstats.Stats(profiler, stream=stream)
        stats.sort_stats('cumulative')
        stats.print_stats(20)
        
        print(f"\nПрофилирование {func.__name__}:")
        print(stream.getvalue())
        
        return result
    return wrapper


# Использование
@profile
def expensive_operation():
    result = 0
    for i in range(1000000):
        result += i ** 2
    return result


@profile
def optimized_operation():
    return sum(i ** 2 for i in range(1000000))
6.2. Профилирование веб-приложения
python
# middleware.py
import cProfile
import io
import pstats
import time
from functools import wraps


def profile_request(func):
    """Декоратор для профилирования эндпоинтов FastAPI/Flask."""
    @wraps(func)
    async def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        
        start = time.time()
        result = await func(*args, **kwargs)
        elapsed = time.time() - start
        
        profiler.disable()
        
        # Сохраняем профиль
        profiler.dump_stats(f'profile_{int(time.time())}.prof')
        
        print(f"Запрос выполнен за {elapsed:.3f} сек")
        
        stream = io.StringIO()
        stats = pstats.Stats(profiler, stream=stream)
        stats.sort_stats('cumulative')
        stats.print_stats(15)
        print(stream.getvalue())
        
        return result
    return wrapper


class ProfilerMiddleware:
    """Middleware для профилирования всех запросов."""
    
    def __init__(self, app, sample_rate: float = 0.1):
        self.app = app
        self.sample_rate = sample_rate
    
    async def __call__(self, scope, receive, send):
        import random
        
        if random.random() < self.sample_rate:
            profiler = cProfile.Profile()
            profiler.enable()
            
            await self.app(scope, receive, send)
            
            profiler.disable()
            profiler.dump_stats(f'profile_{int(time.time())}.prof')
        else:
            await self.app(scope, receive, send)
7. Контрольные вопросы
Что такое профилирование и зачем оно нужно?

В чём разница между cProfile и profile?

Что означают колонки tottime и cumtime в выводе cProfile?

Как запустить профилирование скрипта из командной строки?

Как сохранить результаты профилирования в файл?

Как отсортировать результаты по кумулятивному времени?

Как визуализировать результаты профилирования с помощью snakeviz?

Как найти функцию, которая вызывает больше всего проблем?

Как профилировать отдельную функцию с помощью декоратора?

Чем отличается инструментальное профилирование от статистического?

8. Практическое задание
Задание 1 (базовое)
Напишите функцию, которая вычисляет N-е число Фибоначчи рекурсивно и итеративно. Сравните их производительность с помощью cProfile.

Задание 2 (среднее)
Создайте декоратор @profile, который выводит топ-10 самых медленных функций. Примените его к нескольким функциям.

Задание 3 (сложное)
Профилируйте ETL-скрипт для обработки 100 000 записей. Найдите узкие места и предложите оптимизации.

9. Шпаргалка
bash
# === КОМАНДНАЯ СТРОКА ===
python -m cProfile script.py
python -m cProfile -o output.prof script.py
python -m cProfile -s cumulative script.py

# === КОД ===
import cProfile
profiler = cProfile.Profile()
profiler.enable()
# ваш код
profiler.disable()
profiler.dump_stats('output.prof')

# === АНАЛИЗ ===
import pstats
stats = pstats.Stats('output.prof')
stats.sort_stats('cumulative').print_stats(20)

# === ФИЛЬТРАЦИЯ ===
stats.filter('my_module')
stats.print_callees('function_name')
stats.print_callers('function_name')

# === ВИЗУАЛИЗАЦИЯ ===
snakeviz output.prof
gprof2dot -f pstats output.prof | dot -Tpng -o graph.png
Итог лекции
Вы сегодня:

✅ Изучили основы профилирования с помощью cProfile

✅ Научились анализировать результаты профилирования

✅ Освоили визуализацию результатов с snakeviz и gprof2dot

✅ Научились сравнивать производительность разных реализаций

✅ Применили профилирование в реальных проектах

Теперь вы можете находить и устранять узкие места в вашем коде!
