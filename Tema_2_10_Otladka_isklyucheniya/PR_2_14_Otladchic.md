# ПЗ 2.14. Использование отладчика VS Code. Отладка сложных ошибок

**Тема:** Процесс отладки, точки останова, обработка исключений

**Цель работы:**  
Научиться профессионально использовать отладчик VS Code для поиска и исправления сложных ошибок: логических ошибок, ошибок в циклах, рекурсии, многопоточности. Освоить условные точки останова, точки данных (watch), просмотр стека вызовов.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `pytest`

```bash
pip install pytest
Главная мысль: Отладчик — это скальпель хирурга. print() — это кухонный нож. Для сложной операции выбирайте правильный инструмент.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Возможности отладчика VS Code
Возможность	Горячая клавиша	Описание
Start Debugging	F5	Начать отладку
Step Over	F10	Выполнить строку (не заходя в функции)
Step Into	F11	Войти внутрь функции
Step Out	Shift+F11	Выйти из текущей функции
Continue	F5	Продолжить до следующей точки
Stop	Shift+F5	Остановить отладку
Restart	Ctrl+Shift+F5	Перезапустить отладку
Toggle Breakpoint	F9	Установить/снять точку останова
1.2. Типы точек останова
Тип	Описание	Как создать
Обычная	Останавливается всегда	Клик слева от номера строки
Условная	Останавливается при условии	ПКМ → Edit Breakpoint → условие
Счётная	Останавливается после N раз	ПКМ → Edit Breakpoint → hit count
Лог-точка	Выводит сообщение без остановки	ПКМ → Add Logpoint
Точка данных	Останавливается при изменении переменной	Breakpoints → Data Breakpoint
1.3. Панели отладчика
text
┌─────────────────────────────────────────────────────────────┐
│  ВЫПОЛНЕНИЕ (Call Stack)                                    │
│  ├── calculate()  line 15                                   │
│  ├── process()    line 28                                   │
│  └── main()       line 35                                   │
├─────────────────────────────────────────────────────────────┤
│  ПЕРЕМЕННЫЕ (Variables)                                     │
│  ├── Локальные                                              │
│  │   ├── a: 10                                              │
│  │   ├── b: 0                                               │
│  │   └── result: undefined                                  │
│  └── Глобальные                                             │
│      └── DEBUG: True                                        │
├─────────────────────────────────────────────────────────────┤
│  НАБЛЮДЕНИЕ (Watch)                                         │
│  ├── a * b: 0                                               │
│  └── len(data): 42                                          │
└─────────────────────────────────────────────────────────────┘
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая процесс отладки сложных ошибок.

Техническое задание (нулевой вариант)
Вам дан код с несколькими ошибками. Используя отладчик VS Code, найдите и исправьте все ошибки. Задокументируйте процесс отладки: какие точки останова использовали, какие значения переменных проверяли.

Исходный код с ошибками
python
"""
buggy_code.py — Код с ошибками для отладки.
Задача: найти и исправить 5 ошибок.
"""

import time
from typing import List, Dict


def calculate_average(numbers: List[float]) -> float:
    """
    Вычисляет среднее арифметическое списка чисел.
    ОШИБКА 1: не обрабатывает пустой список
    ОШИБКА 2: неправильное накопление суммы
    """
    total = 0
    for i in range(len(numbers)):
        total += i  # ОШИБКА: нужно numbers[i]
    return total / len(numbers)


def find_maximum(data: List[int]) -> int:
    """
    Находит максимальный элемент в списке.
    ОШИБКА 3: неправильная инициализация
    """
    max_value = 0  # ОШИБКА: если все числа отрицательные
    for item in data:
        if item > max_value:
            max_value = item
    return max_value


def process_users(users: List[Dict]) -> Dict[str, int]:
    """
    Подсчитывает количество пользователей по возрастам.
    ОШИБКА 4: ошибка в условии
    """
    age_groups = {
        "child": 0,    # 0-17
        "adult": 0,    # 18-64
        "senior": 0    # 65+
    }
    
    for user in users:
        age = user.get('age', 0)
        if age < 18:           # ОШИБКА: неправильные границы
            age_groups["child"] += 1
        elif age > 18 and age < 65:  # ОШИБКА: должно быть >=18 и <65
            age_groups["adult"] += 1
        else:
            age_groups["senior"] += 1
    
    return age_groups


def recursive_factorial(n: int) -> int:
    """
    Рекурсивно вычисляет факториал.
    ОШИБКА 5: нет базового случая
    """
    return n * recursive_factorial(n - 1)  # ОШИБКА: бесконечная рекурсия


def main():
    """Главная функция."""
    print("=" * 50)
    print("ТЕСТИРОВАНИЕ ФУНКЦИЙ")
    print("=" * 50)
    
    # Тест 1: calculate_average
    numbers = [10, 20, 30, 40, 50]
    avg = calculate_average(numbers)
    print(f"Среднее {numbers} = {avg}")
    
    # Тест 2: calculate_average с пустым списком
    empty_list = []
    avg_empty = calculate_average(empty_list)  # Должна быть ошибка
    print(f"Среднее пустого списка = {avg_empty}")
    
    # Тест 3: find_maximum
    data = [-5, -2, -10, -1]
    max_val = find_maximum(data)
    print(f"Максимум в {data} = {max_val}")
    
    # Тест 4: process_users
    users = [
        {"name": "Анна", "age": 15},
        {"name": "Борис", "age": 25},
        {"name": "Виктор", "age": 70},
        {"name": "Галина", "age": 18},
    ]
    groups = process_users(users)
    print(f"Возрастные группы: {groups}")
    
    # Тест 5: recursive_factorial
    fact = recursive_factorial(5)
    print(f"Факториал 5 = {fact}")


if __name__ == "__main__":
    main()
Процесс отладки (пошагово)
Шаг 1. Настройка отладчика
Создание launch.json:

json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Отладка buggy_code.py",
            "type": "python",
            "request": "launch",
            "program": "${workspaceFolder}/buggy_code.py",
            "console": "integratedTerminal",
            "justMyCode": false
        }
    ]
}
Шаг 2. Поиск ошибки в calculate_average
Установка точек останова:

Строка 15: total += i

Строка 16: return total / len(numbers)

Процесс отладки:

Нажать F5 для запуска отладки

Программа остановится на точке останова

Посмотреть значения переменных в панели Variables

Обнаружить, что total накапливает индексы, а не значения

Исправление:

python
def calculate_average(numbers: List[float]) -> float:
    if not numbers:  # Обработка пустого списка
        return 0.0
    total = 0
    for num in numbers:  # Исправлено: итерация по значениям
        total += num
    return total / len(numbers)
Шаг 3. Поиск ошибки в find_maximum
Установка точек останова:

Строка 28: max_value = 0

Строка 30: if item > max_value

Процесс отладки:

Запустить отладку с F5

Наблюдать, что max_value инициализирован 0

При сравнении -1 > 0 — False, максимум не обновляется

Возвращается 0, которого нет в списке

Исправление:

python
def find_maximum(data: List[int]) -> int:
    if not data:
        raise ValueError("Список не может быть пустым")
    max_value = data[0]  # Инициализация первым элементом
    for item in data[1:]:
        if item > max_value:
            max_value = item
    return max_value
Шаг 4. Поиск ошибки в process_users
Установка условной точки останова:

Строка 51: if age < 18 — условие age == 18

Процесс отладки:

Правая кнопка мыши по точке останова → Edit Breakpoint

Ввести условие age == 18

Запустить отладку

При age = 18 программа остановится

Обнаружить, что 18 попадает в else (senior)

Исправление:

python
def process_users(users: List[Dict]) -> Dict[str, int]:
    age_groups = {
        "child": 0,    # 0-17
        "adult": 0,    # 18-64
        "senior": 0    # 65+
    }
    
    for user in users:
        age = user.get('age', 0)
        if age < 18:
            age_groups["child"] += 1
        elif 18 <= age < 65:  # Исправлено
            age_groups["adult"] += 1
        else:
            age_groups["senior"] += 1
    
    return age_groups
Шаг 5. Поиск ошибки в recursive_factorial
Установка точки останова:

Строка 65: return n * recursive_factorial(n - 1)

Процесс отладки:

Запустить отладку

Наблюдать в Call Stack, что функция вызывает сама себя бесконечно

Стек вызовов растёт, пока не произойдёт RecursionError

Исправление:

python
def recursive_factorial(n: int) -> int:
    if n <= 1:  # Базовый случай
        return 1
    return n * recursive_factorial(n - 1)
Исправленный код
python
"""
fixed_code.py — Исправленная версия.
"""

from typing import List, Dict


def calculate_average(numbers: List[float]) -> float:
    """Вычисляет среднее арифметическое списка чисел."""
    if not numbers:
        return 0.0
    total = 0
    for num in numbers:
        total += num
    return total / len(numbers)


def find_maximum(data: List[int]) -> int:
    """Находит максимальный элемент в списке."""
    if not data:
        raise ValueError("Список не может быть пустым")
    max_value = data[0]
    for item in data[1:]:
        if item > max_value:
            max_value = item
    return max_value


def process_users(users: List[Dict]) -> Dict[str, int]:
    """Подсчитывает количество пользователей по возрастам."""
    age_groups = {
        "child": 0,    # 0-17
        "adult": 0,    # 18-64
        "senior": 0    # 65+
    }
    
    for user in users:
        age = user.get('age', 0)
        if age < 18:
            age_groups["child"] += 1
        elif 18 <= age < 65:
            age_groups["adult"] += 1
        else:
            age_groups["senior"] += 1
    
    return age_groups


def recursive_factorial(n: int) -> int:
    """Рекурсивно вычисляет факториал."""
    if n <= 1:
        return 1
    return n * recursive_factorial(n - 1)


def main():
    """Главная функция."""
    print("=" * 50)
    print("ТЕСТИРОВАНИЕ ФУНКЦИЙ (ИСПРАВЛЕННЫХ)")
    print("=" * 50)
    
    # Тест 1: calculate_average
    numbers = [10, 20, 30, 40, 50]
    avg = calculate_average(numbers)
    print(f"Среднее {numbers} = {avg}")
    
    # Тест 2: calculate_average с пустым списком
    empty_list = []
    avg_empty = calculate_average(empty_list)
    print(f"Среднее пустого списка = {avg_empty}")
    
    # Тест 3: find_maximum
    data = [-5, -2, -10, -1]
    max_val = find_maximum(data)
    print(f"Максимум в {data} = {max_val}")
    
    # Тест 4: process_users
    users = [
        {"name": "Анна", "age": 15},
        {"name": "Борис", "age": 25},
        {"name": "Виктор", "age": 70},
        {"name": "Галина", "age": 18},
    ]
    groups = process_users(users)
    print(f"Возрастные группы: {groups}")
    
    # Тест 5: recursive_factorial
    fact = recursive_factorial(5)
    print(f"Факториал 5 = {fact}")


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
==================================================
ТЕСТИРОВАНИЕ ФУНКЦИЙ (ИСПРАВЛЕННЫХ)
==================================================
Среднее [10, 20, 30, 40, 50] = 30.0
Среднее пустого списка = 0.0
Максимум в [-5, -2, -10, -1] = -1
Возрастные группы: {'child': 1, 'adult': 2, 'senior': 1}
Факториал 5 = 120
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (3-4 ошибки, простые функции)

Варианты 9-17: средний (5-6 ошибок, работа с данными)

Варианты 18-25: сложный (7+ ошибок, рекурсия, многопоточность)

Варианты 1-8 (Базовый уровень)
№	Тип ошибок	Код для отладки
1	Арифметические, циклы	Калькулятор среднего, суммы, произведения
2	Условные операторы	Проверка возраста, категории
3	Списки и индексы	Поиск элемента, сортировка
4	Строки	Конкатенация, форматирование
5	Словари	Доступ к ключам, обновление
6	Функции	Передача параметров, возврат значений
7	Ввод-вывод	Чтение из файла, преобразование типов
8	Математические	Площадь фигур, округление
Пример варианта 1:

python
# variant_1.py
def calculate_average(numbers):
    total = 0
    for i in range(len(numbers)):
        total += i
    return total / len(numbers)

def calculate_sum(numbers):
    sum = 0
    for num in numbers:
        sum = num
    return sum

def main():
    data = [10, 20, 30]
    print(f"Среднее: {calculate_average(data)}")
    print(f"Сумма: {calculate_sum(data)}")
Варианты 9-17 (Средний уровень)
№	Тип ошибок	Дополнительные элементы
9	Рекурсия	Факториал, Фибоначчи
10	Вложенные циклы	Таблица умножения, матрицы
11	Обработка исключений	Отсутствие try-except
12	Работа с файлами	Неправильное открытие/закрытие
13	Дата и время	Неправильный формат, часовые пояса
14	Классы	Конструкторы, методы, атрибуты
15	Наследование	Вызов parent методов
16	Списки (сложные)	Срезы, копирование, ссылки
17	Генераторы	Yield, итерация
Пример варианта 9:

python
# variant_9.py
def factorial(n):
    return n * factorial(n - 1)

def fibonacci(n):
    if n <= 0:
        return 0
    elif n == 1:
        return 1
    else:
        return fibonacci(n) + fibonacci(n - 1)

def main():
    print(f"Факториал 5: {factorial(5)}")
    print(f"Fibonacci 10: {fibonacci(10)}")
Варианты 18-25 (Сложный уровень)
№	Тип ошибок	Специфика
18	Многопоточность	Гонки данных, deadlock
19	Асинхронность	await, async, корутины
20	Декораторы	Порядок выполнения, аргументы
21	Контекстные менеджеры	enter, exit
22	Метаклассы	Создание классов
23	Свойства (property)	Геттеры, сеттеры
24	Дескрипторы	get, set
25	Cython/Расширения	Интеграция с C
Пример варианта 18:

python
# variant_18.py
import threading

counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1

def decrement():
    global counter
    for _ in range(100000):
        counter -= 1

def main():
    t1 = threading.Thread(target=increment)
    t2 = threading.Thread(target=decrement)
    t1.start()
    t2.start()
    t1.join()
    t2.join()
    print(f"Counter: {counter}")  # Должен быть 0, но из-за гонок данных — нет
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Ошибки не найдены, код не исправлен
3 (удовлетворительно)	Найдены 2-3 ошибки, но не все исправлены
4 (хорошо)	Найдены все ошибки, код исправлен, есть описание процесса
5 (отлично)	Все ошибки найдены и исправлены, задокументирован процесс отладки (скриншоты, описание шагов)
5. Шпаргалка
5.1. Горячие клавиши отладчика
Действие	Windows/Linux	macOS
Начать отладку	F5	F5
Шаг с обходом	F10	F10
Шаг с заходом	F11	F11
Шаг с выходом	Shift+F11	Shift+F11
Продолжить	F5	F5
Остановить	Shift+F5	Shift+F5
Перезапустить	Ctrl+Shift+F5	Ctrl+Shift+F5
Точка останова	F9	F9
5.2. Условные точки останова
python
# Синтаксис условий для точек останова
i == 10              # остановиться при i == 10
len(data) > 100      # остановиться при длине > 100
user.get('age') > 18 # остановиться при возрасте > 18
5.3. Команды отладчика в консоли (pdb)
python
import pdb; pdb.set_trace()  # программная точка останова

# Команды pdb в консоли:
# n (next)     - выполнить строку
# s (step)     - войти в функцию
# c (continue) - продолжить
# p var        - напечатать переменную
# q (quit)     - выйти
5.4. Шаблон отчёта об отладке
markdown
## Отчёт по отладке

### Ошибка №1
- **Место:** строка X, функция Y
- **Симптом:** программа выдаёт неверный результат
- **Как нашли:** установили точку останова, проверили переменные
- **Причина:** неправильная инициализация переменной
- **Исправление:** заменили `value = 0` на `value = data[0]`

### Ошибка №2
...
Карточка студента
text
ПЗ 2.14. ИСПОЛЬЗОВАНИЕ ОТЛАДЧИКА VS CODE

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== НАЙДЕННЫЕ ОШИБКИ ===

Ошибка 1: _________________________________
Ошибка 2: _________________________________
Ошибка 3: _________________________________
Ошибка 4: _________________________________
Ошибка 5: _________________________________

=== ИСПОЛЬЗОВАННЫЕ ИНСТРУМЕНТЫ ===

□ Точки останова (какие строки): _________
□ Условные точки (условия): ______________
□ Лог-точки: _____________________________
□ Панель Watch (какие выражения): ________
□ Call Stack (какие вызовы): _____________

=== РЕЗУЛЬТАТ ===

Исправленный код прилагается
Скриншоты процесса отладки прилагаются

Дата выполнения: _____________
