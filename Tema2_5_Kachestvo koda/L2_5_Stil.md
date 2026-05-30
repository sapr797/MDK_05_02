# Тема 2.5. Качество кода

## Лекция 2.5. Стиль программирования: PEP 8. Статический анализ кода: pylint, radon

**Цель лекции:**  
Научиться писать код, который соответствует профессиональным стандартам, автоматически проверять качество кода с помощью статических анализаторов, понимать метрики сложности и поддерживаемости кода.

> Главная мысль: **Хороший код — это не тот, который работает. Хороший код — тот, который понятен другому разработчику (и вам самому через полгода).**

---

## Часть 1. PEP 8 — стандарт стиля Python

### 1.1. Что такое PEP 8 и зачем он нужен

**PEP** (Python Enhancement Proposal) — документ с предложениями по улучшению Python.  
**PEP 8** — это руководство по написанию читаемого кода.

**Почему важно соблюдать PEP 8:**
- Единый стиль → код легче читать и поддерживать
- Автоматические инструменты (линтеры) помогают находить ошибки
- Работодатели проверяют знание PEP 8 на собеседованиях

### 1.2. Основные правила PEP 8 (с примерами)

#### Правило 1: Отступы (4 пробела)

**Плохо:**
```python
def func():
  return True   # 2 пробела
Хорошо:

python
def func():
    return True   # 4 пробела
Правило 2: Максимальная длина строки — 79 символов
Плохо:

python
result = some_very_long_function_name(with_many_arguments, and_even_more, and_this_is_definitely_more_than_79_characters_in_one_line)
Хорошо:

python
result = some_very_long_function_name(
    with_many_arguments, 
    and_even_more, 
    and_this_is_split_across_lines
)
Правило 3: Именование переменных, функций, классов
Тип	Стиль	Пример
Переменная	snake_case	user_name, total_count
Функция	snake_case	get_user_data(), calculate_sum()
Класс	PascalCase	UserProfile, DataProcessor
Константа	UPPER_SNAKE_CASE	MAX_RETRIES, API_KEY
Приватный атрибут	_snake_case	_internal_value
Правило 4: Пробелы вокруг операторов
Плохо:

python
x=5
result = a+b
if x==5:
Хорошо:

python
x = 5
result = a + b
if x == 5:
Правило 5: Пустые строки
2 пустые строки между функциями на верхнем уровне

1 пустая строка между методами в классе

python
def first_function():
    pass


def second_function():
    pass


class MyClass:
    
    def first_method(self):
        pass
    
    def second_method(self):
        pass
Правило 6: Импорты — в начале файла, сгруппированы
python
# 1. Стандартная библиотека
import os
import sys
from datetime import datetime

# 2. Сторонние библиотеки
import requests
import numpy as np

# 3. Локальные модули
from myproject.models import User
from myproject.utils import helper
Правило 7: Сравнение с None
Плохо:

python
if value == None:
if value != None:
Хорошо:

python
if value is None:
if value is not None:
1.3. Как проверить код на соответствие PEP 8
Встроенные инструменты VS Code:

Расширение Python автоматически подсвечивает нарушения

Shift+Alt+F — автоформатирование (если настроен форматтер)

Командная строка:

bash
# Установка autopep8
pip install autopep8

# Автоисправление файла
autopep8 --in-place --aggressive --aggressive myfile.py

# Проверка без исправления
pycodestyle myfile.py
Часть 2. Статический анализ кода
2.1. Что такое статический анализ
Статический анализ — проверка кода без его выполнения. Анализаторы находят:

Синтаксические ошибки

Нарушения стиля (PEP 8)

Потенциальные баги

Неиспользуемые переменные и импорты

Сложность кода

2.2. Pylint — главный линтер для Python
Установка:

bash
pip install pylint
Запуск:

bash
pylint myfile.py
pylint src/                 # проверить всю папку
Пример вывода Pylint:

text
************* Module myfile
myfile.py:1:0: C0114: Missing module docstring (missing-module-docstring)
myfile.py:3:0: C0103: Function name 'bad' doesn't conform to snake_case naming style (invalid-name)
myfile.py:5:4: W0612: Unused variable 'x' (unused-variable)

------------------------------------------------------------------
Your code has been rated at 6.67/10
Что означают коды:

Код	Тип	Описание
C	Convention	Нарушение стиля (PEP 8)
R	Refactor	Плохой запах кода (можно улучшить)
W	Warning	Потенциальная проблема
E	Error	Ошибка (код не выполнится)
F	Fatal	Фатальная ошибка Pylint
Оценка Pylint:

10/10 — идеальный код

8-9/10 — хороший код

6-7/10 — удовлетворительный

Ниже 6 — требуется рефакторинг

2.3. Настройка Pylint
Создайте файл .pylintrc в корне проекта:

ini
[MASTER]
# Игнорировать файлы
ignore=tests,venv

[MESSAGES CONTROL]
# Отключить некоторые проверки
disable=C0114,  # missing-module-docstring
        C0115,  # missing-class-docstring
        C0116,  # missing-function-docstring
        R0903,  # too-few-public-methods
        R0201   # no-self-use

[FORMAT]
# Максимальная длина строки
max-line-length=100

[BASIC]
# Допустимые имена переменных
good-names=i,j,k,ex,Run,_
Запуск с конфигом:

bash
pylint --rcfile=.pylintrc src/
2.4. Radon — анализ сложности кода
Radon вычисляет метрики сложности кода.

Установка:

bash
pip install radon
Основные команды:

bash
# Цикломатическая сложность (насколько код сложен для понимания)
radon cc myfile.py

# Показать только сложные функции (ранг C и выше)
radon cc -s -a myfile.py

# Индекс поддерживаемости (Maintainability Index)
radon mi myfile.py

# Сырые метрики (строки кода, комментарии)
radon raw myfile.py

# Сложность в виде HTML-отчёта
radon cc --json myfile.py > complexity.json
Шкала цикломатической сложности (radon cc):

Ранг	Сложность	Оценка
A	1-5	Просто, легко тестировать
B	6-10	Хорошо
C	11-20	Сложно, требует рефакторинга
D	21-30	Очень сложно, высокий риск
E	31-40	Критически сложно
F	41+	Неподдерживаемо
Пример вывода:

text
$ radon cc -s calculator.py
calculator.py
    F 1:0 calculate - B (11)
    F 5:2 parse_expression - C (15)
    F 12:4 evaluate - A (4)
2.5. Другие инструменты статического анализа
Инструмент	Назначение
flake8	Лёгкий линтер (PEP 8 + питонические ошибки)
black	Бескомпромиссный форматтер (единый стиль)
isort	Сортировка импортов
mypy	Проверка типов (аннотации)
bandit	Поиск уязвимостей безопасности
pre-commit	Запуск проверок перед коммитом
Часть 3. Практический пример: от "плохого" к "хорошему" коду
3.1. Исходный "плохой" код
python
# bad_code.py
import math, sys, os, json
def f(x,y):
    if x>0 and y>0:return math.sqrt(x*y)
    else:
        return None
def calculate(values):
    result=0
    for i in range(len(values)):
        result+=values[i]
    return result
class dataProcessor:
    def __init__(self,data):
        self.data=data
    def process(self):
        d={}
        for item in self.data:
            if item not in d:
                d[item]=1
            else:
                d[item]+=1
        return d
def main():
    x=10
    y=5
    res=f(x,y)
    print(res)
3.2. Запуск статического анализа
bash
# Проверка PEP 8
pycodestyle bad_code.py

# Вывод:
# bad_code.py:2:21: E231 missing whitespace after ','
# bad_code.py:2:23: E231 missing whitespace after ','
# bad_code.py:3:0: E302 expected 2 blank lines, found 0
# bad_code.py:3:5: E701 multiple statements on one line
# bad_code.py:4:19: E711 comparison to None should be 'if cond is None:'
# bad_code.py:7:13: E225 missing whitespace around operator
# ...

# Проверка Pylint
pylint bad_code.py --exit-zero
# Rating: 2.33/10
3.3. Исправленный код (по PEP 8 и рекомендациям)
python
# good_code.py
"""
Модуль с математическими утилитами и обработкой данных.

Содержит функции для вычислений и класс для анализа частоты элементов.
"""

import json
import math
import os
import sys
from typing import List, Dict, Optional


def geometric_mean(x: float, y: float) -> Optional[float]:
    """
    Вычисляет среднее геометрическое двух положительных чисел.

    Args:
        x: Первое положительное число
        y: Второе положительное число

    Returns:
        Среднее геометрическое или None, если числа не положительные
    """
    if x > 0 and y > 0:
        return math.sqrt(x * y)
    return None


def calculate_sum(values: List[float]) -> float:
    """
    Вычисляет сумму элементов списка.

    Args:
        values: Список чисел

    Returns:
        Сумма элементов
    """
    return sum(values)


class DataProcessor:
    """Класс для обработки и анализа данных."""

    def __init__(self, data: List):
        """
        Инициализирует процессор данных.

        Args:
            data: Список данных для обработки
        """
        self.data = data

    def process(self) -> Dict:
        """
        Подсчитывает частоту вхождения каждого элемента.

        Returns:
            Словарь с элементами и их частотой
        """
        frequency = {}
        for item in self.data:
            frequency[item] = frequency.get(item, 0) + 1
        return frequency


def main() -> None:
    """Основная функция программы."""
    x = 10
    y = 5
    result = geometric_mean(x, y)
    print(result)


if __name__ == "__main__":
    main()
3.4. Результаты после исправления
bash
pylint good_code.py
# Your code has been rated at 9.67/10

radon cc good_code.py
# good_code.py
#     F 11:0 geometric_mean - A (2)
#     F 28:0 calculate_sum - A (1)
#     F 44:0 __init__ - A (1)
#     F 52:0 process - A (2)
#     F 66:0 main - A (1)
Часть 4. Автоматизация проверок
4.1. Настройка pre-commit hooks
Установка pre-commit:

bash
pip install pre-commit
Создание .pre-commit-config.yaml:

yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.9.0
    hooks:
      - id: black
        language_version: python3

  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort

  - repo: https://github.com/pycqa/flake8
    rev: 6.1.0
    hooks:
      - id: flake8
        args: [--max-line-length=88]

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.5.1
    hooks:
      - id: mypy

  - repo: https://github.com/pycqa/pylint
    rev: v3.0.0
    hooks:
      - id: pylint
        args: [--rcfile=.pylintrc]
Установка хуков:

bash
pre-commit install
Теперь при каждом git commit автоматически запускаются проверки.

4.2. Настройка VS Code для автоматической проверки
Расширения для VS Code:

Python (Microsoft) — базовая поддержка

Pylint — интеграция линтера

Black Formatter — автоформатирование

isort — сортировка импортов

Настройки settings.json:

json
{
    "[python]": {
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.organizeImports": true
        },
        "editor.defaultFormatter": "ms-python.black-formatter"
    },
    "python.linting.enabled": true,
    "python.linting.pylintEnabled": true,
    "python.linting.flake8Enabled": false,
    "python.linting.mypyEnabled": true
}
Часть 5. Метрики качества кода
5.1. Ключевые метрики
Метрика	Описание	Хорошее значение
LOC (Lines of Code)	Количество строк кода	< 400 на файл
Cyclomatic Complexity	Количество независимых путей	< 10 на функцию
Maintainability Index	Лёгкость поддержки	> 65
Code Coverage	Покрытие тестами	> 80%
Comment Ratio	Доля комментариев	15-25%
5.2. Формула индекса поддерживаемости (Halstead)
text
MI = 171 - 5.2 * ln(V) - 0.23 * G - 16.2 * ln(LOC)
где:

V — объём кода (Halstead Volume)

G — цикломатическая сложность

LOC — строки кода

Интерпретация:

85-100: отлично

65-84: хорошо

20-64: удовлетворительно

0-19: требуется рефакторинг

5.3. Проверка метрик в CI/CD
Пример GitLab CI:

yaml
pylint-job:
  stage: test
  script:
    - pip install pylint radon
    - pylint src/ --fail-under=8.0
    - radon cc src/ -a -nc
    - radon mi src/ -s
  only:
    - merge_requests
Шпаргалка по инструментам
bash
# === УСТАНОВКА ===
pip install pylint radon black isort flake8 mypy pre-commit

# === PYLINT ===
pylint myfile.py                          # проверить файл
pylint src/ --fail-under=8.0              # проверка с требованием
pylint --generate-rcfile > .pylintrc      # создать конфиг

# === RADON ===
radon cc src/ -s -a                       # сложность
radon mi src/ -s                          # поддерживаемость
radon raw src/                            # сырые метрики

# === ФОРМАТТЕРЫ ===
black myfile.py                           # автоформатирование
black --check myfile.py                   # проверить, не меняя
isort myfile.py                           # сортировка импортов

# === ПРОВЕРКА СТИЛЯ ===
flake8 myfile.py                          # PEP 8 проверка
pycodestyle myfile.py                     # альтернатива

# === ПРОВЕРКА ТИПОВ ===
mypy myfile.py                            # проверка аннотаций

# === PRE-COMMIT ===
pre-commit install                        # установка хуков
pre-commit run --all-files                # проверить все файлы
Контрольные вопросы
Зачем нужен PEP 8? Какие 3 основных правила вы запомнили?

Чем отличается Pylint от Radon? Что измеряет каждый?

Что означает цикломатическая сложность? Какое значение считается хорошим?

Как настроить автоматическую проверку кода при каждом сохранении в VS Code?

Что будет, если у функции сложность = 25? Как это исправить?

Зачем нужен .pylintrc? Что можно в нём настроить?

Какие метрики включает индекс поддерживаемости (Maintainability Index)?

Что произойдёт, если закоммитить код, не прошедший pre-commit проверки?

Практическое задание (для закрепления)
Задание: Возьмите любой свой старый проект (или код варианта из ПЗ 2.4) и выполните:

Запустите pylint и зафиксируйте оценку

Запустите radon cc — найдите самую сложную функцию

Исправьте код в соответствии с PEP 8

Добейтесь оценки Pylint не ниже 8.0

Настройте pre-commit для проекта

Создайте отчёт с "до" и "после"

Итог лекции
Вы сегодня:

Изучили основные правила PEP 8

Научились пользоваться Pylint для статического анализа

Освоили Radon для оценки сложности кода

Узнали об автоматизации проверок через pre-commit

Поняли метрики качества кода

Теперь ваш код будет не только работать, но и радовать глаз других разработчиков!

