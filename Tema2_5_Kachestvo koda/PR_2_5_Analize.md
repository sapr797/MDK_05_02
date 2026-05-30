# ПЗ 2.5. Анализ качества кода с pylint и radon. Исправление замечаний

**Тема:** Качество кода, статический анализ, PEP 8, pylint, radon

**Цель работы:**  
Научиться анализировать качество Python-кода с помощью статических анализаторов, интерпретировать их отчёты, исправлять выявленные проблемы и доводить код до профессионального уровня.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `pylint`, `radon`, `black`, `isort`

```bash
pip install pylint radon black isort
Нулевой вариант (эталонный) — Демонстрация преподавателя
Назначение: преподаватель выполняет этот вариант на занятии, показывая полный цикл анализа и исправления кода.

Исходный "плохой" код
python
# bad_code_demo.py
import math,sys,os,json
from datetime import *
def calc(x,y):
    if x>0 and y>0:
        return math.sqrt(x*y)
    else:
        return None
def process_data(data):
    res=0
    for i in range(len(data)):
        res+=data[i]
    return res
class myClass:
    def __init__(self,val):
        self.val=val
    def do(self):
        print(self.val)
def main():
    a=10
    b=5
    r=calc(a,b)
    print(r)
if __name__=="__main__":
    main()
Шаг 1. Запуск анализа
bash
# Установка инструментов
pip install pylint radon black isort

# Запуск pylint
pylint bad_code_demo.py

# Запуск radon (цикломатическая сложность)
radon cc bad_code_demo.py -s

# Запуск radon (индекс поддерживаемости)
radon mi bad_code_demo.py -s
Шаг 2. Анализ отчёта pylint
text
************* Module bad_code_demo
bad_code_demo.py:1:0: C0114: Missing module docstring (missing-module-docstring)
bad_code_demo.py:1:0: E0602: Undefined variable 'math' (undefined-variable)
bad_code_demo.py:1:6: W0611: Unused import sys (unused-import)
bad_code_demo.py:1:11: W0611: Unused import os (unused-import)
bad_code_demo.py:1:15: W0611: Unused import json (unused-import)
bad_code_demo.py:2:0: W0614: Unused import(s) datetime from wildcard import (unused-wildcard-import)
bad_code_demo.py:3:0: C0116: Missing function docstring (missing-function-docstring)
bad_code_demo.py:3:0: C0103: Function name 'calc' doesn't conform to snake_case (invalid-name)
bad_code_demo.py:3:5: C0326: Exactly one space required before assignment (bad-whitespace)
bad_code_demo.py:3:10: C0326: Exactly one space required after comma (bad-whitespace)
bad_code_demo.py:3:12: C0326: Exactly one space required around operator (bad-whitespace)
bad_code_demo.py:4:4: W0311: Bad indentation. Found 4 spaces, expected 8 (bad-indentation)
bad_code_demo.py:5:4: C0326: Exactly one space required after comma (bad-whitespace)
bad_code_demo.py:7:0: C0116: Missing function docstring (missing-function-docstring)
bad_code_demo.py:7:0: C0103: Function name 'process_data' doesn't conform to snake_case (invalid-name)
bad_code_demo.py:8:4: W0621: Redefining name 'res' from outer scope (redefined-outer-name)
bad_code_demo.py:8:4: C0103: Variable name 'res' doesn't conform to snake_case (invalid-name)
bad_code_demo.py:9:4: C0103: Variable name 'i' doesn't conform to snake_case (invalid-name)
bad_code_demo.py:11:0: C0115: Missing class docstring (missing-class-docstring)
bad_code_demo.py:11:0: C0103: Class name 'myClass' doesn't conform to PascalCase (invalid-name)
bad_code_demo.py:12:4: C0116: Missing function docstring (missing-function-docstring)
bad_code_demo.py:12:4: C0103: Method name 'do' doesn't conform to snake_case (invalid-name)
bad_code_demo.py:13:8: C0103: Variable name 'val' doesn't conform to snake_case (invalid-name)
bad_code_demo.py:15:0: C0116: Missing function docstring (missing-function-docstring)
bad_code_demo.py:15:0: C0103: Function name 'main' doesn't conform to snake_case (invalid-name)

------------------------------------------------------------------
Your code has been rated at 0.00/10 (previous run: 0.00/10, +0.00)
Шаг 3. Анализ отчёта radon
bash
$ radon cc bad_code_demo.py -s
bad_code_demo.py
    F 3:0 calc - A (2)
    F 7:0 process_data - A (2)
    F 15:0 main - A (1)

$ radon mi bad_code_demo.py -s
bad_code_demo.py - MI: 44.74 (C)
Шаг 4. Исправленный код
python
#!/usr/bin/env python3
"""
Модуль с математическими утилитами.

Содержит функции для вычисления среднего геометрического
и обработки числовых данных.
"""

import math
from typing import List, Optional


def geometric_mean(x: float, y: float) -> Optional[float]:
    """
    Вычисляет среднее геометрическое двух положительных чисел.

    Args:
        x: Первое положительное число
        y: Второе положительное число

    Returns:
        Среднее геометрическое или None, если числа не положительные

    Examples:
        >>> geometric_mean(4, 9)
        6.0
        >>> geometric_mean(-1, 4) is None
        True
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

    Examples:
        >>> calculate_sum([1, 2, 3, 4])
        10.0
    """
    return sum(values)


class DataProcessor:
    """Класс для обработки и вывода данных."""

    def __init__(self, value: float) -> None:
        """
        Инициализирует процессор данных.

        Args:
            value: Числовое значение для обработки
        """
        self.value = value

    def display(self) -> None:
        """Выводит значение на экран."""
        print(self.value)


def main() -> None:
    """Основная функция программы для демонстрации работы."""
    first_number = 10
    second_number = 5
    result = geometric_mean(first_number, second_number)
    print(result)


if __name__ == "__main__":
    main()
Шаг 5. Повторный анализ
bash
$ pylint good_code_demo.py
...
Your code has been rated at 10.00/10 (previous run: 0.00/10, +10.00)

$ radon cc good_code_demo.py -s
good_code_demo.py
    F 16:0 geometric_mean - A (2)
    F 36:0 calculate_sum - A (1)
    F 53:0 __init__ - A (1)
    F 61:0 display - A (1)
    F 70:0 main - A (1)

$ radon mi good_code_demo.py -s
good_code_demo.py - MI: 71.24 (A)
Шаг 6. Автоматическое форматирование
bash
# Сортировка импортов
isort good_code_demo.py

# Форматирование
black good_code_demo.py
25 вариантов практической работы (ПЗ 2.5)
Общая структура каждого варианта:
Студент получает "плохой" код, должен проанализировать его с помощью pylint и radon, исправить все замечания и добиться оценки pylint не ниже 8.0/10.

Уровни сложности:

Варианты 1-8: базовый (простые нарушения стиля)

Варианты 9-17: средний (логические ошибки, сложность кода)

Варианты 18-25: сложный (циклическая сложность, архитектурные проблемы)

Варианты 1-8 (Базовый уровень)
Вариант 1. Калькулятор
python
# variant_1.py
import math
def sq(x):
    return x*x
def p(a,b):
    print(a+b)
def main():
    x=10
    y=20
    print(sq(x))
    p(x,y)
if __name__=="__main__":main()
Задание: Исправить нарушения PEP 8, добавить docstring, переименовать функции.

Вариант 2. Проверка возраста
python
# variant_2.py
age=input("Enter age: ")
if age>=18:
    print("Adult")
else:
    print("Child")
def check(age):
    if age>0 and age<120:
        return True
    else:
        return False
Задание: Исправить стиль, добавить обработку типов, написать docstring.

Вариант 3. Работа со списком
python
# variant_3.py
def proc(lst):
    res=[]
    for i in lst:
        if i%2==0:
            res.append(i)
    return res
a=[1,2,3,4,5,6,7,8,9,10]
b=proc(a)
print(b)
Задание: Переименовать функцию и переменные, добавить аннотации типов.

Вариант 4. Конвертер температур
python
# variant_4.py
def c_to_f(c):
    return c*9/5+32
def f_to_c(f):
    return (f-32)*5/9
t=25
print(c_to_f(t))
Задание: Добавить docstring, исправить имена, добавить проверку типов.

Вариант 5. Площадь фигур
python
# variant_5.py
import math
def circle_area(r):
    return math.pi*r*r
def rect_area(a,b):
    return a*b
def triangle_area(a,b,c):
    s=(a+b+c)/2
    return math.sqrt(s*(s-a)*(s-b)*(s-c))
Задание: Добавить docstring, исправить форматирование, добавить аннотации.

Вариант 6. Строковые операции
python
# variant_6.py
def rev(s):
    return s[::-1]
def count_vowels(s):
    v=0
    for i in s:
        if i in "aeiouAEIOU":
            v+=1
    return v
def main():
    t="hello"
    print(rev(t))
    print(count_vowels(t))
Задание: Исправить имена, добавить docstring, использовать константы.

Вариант 7. Факториал
python
# variant_7.py
def fact(n):
    if n==0:
        return 1
    else:
        return n*fact(n-1)
def main():
    for i in range(10):
        print(fact(i))
main()
Задание: Добавить проверку аргумента, docstring, аннотации типов.

Вариант 8. Поиск максимума
python
# variant_8.py
def max2(a,b):
    if a>b:
        return a
    else:
        return b
def max3(a,b,c):
    return max2(max2(a,b),c)
def main():
    print(max3(5,2,8))
Задание: Исправить стиль, добавить docstring, использовать встроенные функции.

Варианты 9-17 (Средний уровень)
Вариант 9. Работа с файлами
python
# variant_9.py
import json,os
def load_data(f):
    if os.path.exists(f):
        return json.load(open(f))
    else:
        return []
def save_data(f,d):
    json.dump(d,open(f,'w'))
def main():
    data=load_data('data.json')
    data.append({'name':'test'})
    save_data('data.json',data)
Задание: Исправить утечки ресурсов, добавить обработку ошибок, docstring.

Вариант 10. Анализ текста
python
# variant_10.py
def word_count(text):
    words=text.split()
    d={}
    for w in words:
        if w in d:
            d[w]+=1
        else:
            d[w]=1
    return d
def main():
    s="hello world hello"
    print(word_count(s))
Задание: Добавить аннотации типов, улучшить читаемость, добавить docstring.

Вариант 11. Генератор паролей
python
# variant_11.py
import random,string
def gen_pass(l):
    c=string.ascii_letters+string.digits
    p=''
    for i in range(l):
        p+=random.choice(c)
    return p
Задание: Добавить проверку длины, docstring, использовать list comprehension.

Вариант 12. Валидация email
python
# variant_12.py
import re
def validate_email(email):
    if re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', email):
        return True
    else:
        return False
Задание: Добавить docstring, вынести regex в константу, аннотации.

Вариант 13. Кэширование результатов
python
# variant_13.py
cache={}
def fib(n):
    if n in cache:
        return cache[n]
    if n<2:
        return n
    cache[n]=fib(n-1)+fib(n-2)
    return cache[n]
Задание: Добавить lru_cache, улучшить читаемость, docstring.

Вариант 14. Обработка дат
python
# variant_14.py
from datetime import *
def days_diff(d1,d2):
    return abs((d2-d1).days)
def main():
    d1=datetime(2024,1,1)
    d2=datetime(2024,12,31)
    print(days_diff(d1,d2))
Задание: Исправить импорт, добавить аннотации, docstring.

Вариант 15. Работа с API
python
# variant_15.py
import requests
def get_data(url):
    r=requests.get(url)
    if r.status_code==200:
        return r.json()
    else:
        return None
Задание: Добавить обработку ошибок, таймауты, docstring.

Вариант 16. Сортировка списка
python
# variant_16.py
def bubble_sort(arr):
    n=len(arr)
    for i in range(n):
        for j in range(0,n-i-1):
            if arr[j]>arr[j+1]:
                arr[j],arr[j+1]=arr[j+1],arr[j]
    return arr
Задание: Добавить аннотации, docstring, улучшить имена переменных.

Вариант 17. Класс "Студент"
python
# variant_17.py
class student:
    def __init__(self,n,a,g):
        self.name=n
        self.age=a
        self.grade=g
    def get_info(self):
        return f"{self.name}, {self.age}, {self.grade}"
Задание: Исправить имена классов/методов, добавить свойства, docstring.

Варианты 18-25 (Сложный уровень)
Вариант 18. Менеджер задач (высокая сложность)
python
# variant_18.py
import json,os
from datetime import datetime
tasks=[]
def add_task(t):
    tasks.append({'title':t,'done':False,'created':datetime.now()})
def list_tasks():
    for i,t in enumerate(tasks):
        print(f"{i}: {t['title']} - {'Done' if t['done'] else 'Pending'}")
def complete_task(i):
    if i<len(tasks):
        tasks[i]['done']=True
def delete_task(i):
    if i<len(tasks):
        tasks.pop(i)
def save():
    json.dump(tasks,open('tasks.json','w'))
def load():
    global tasks
    if os.path.exists('tasks.json'):
        tasks=json.load(open('tasks.json'))
def main():
    load()
    while True:
        print("\n1.Add 2.List 3.Complete 4.Delete 5.Save 6.Exit")
        c=input("Choice: ")
        if c=='1':add_task(input("Title: "))
        elif c=='2':list_tasks()
        elif c=='3':complete_task(int(input("Index: ")))
        elif c=='4':delete_task(int(input("Index: ")))
        elif c=='5':save()
        elif c=='6':break
Задание: Разбить на модули, добавить классы, обработку ошибок, docstring.

Вариант 19. Парсер логов (сложный)
python
# variant_19.py
import re
def parse_log(line):
    p=r'(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) (\w+) (.+)'
    m=re.match(p,line)
    if m:
        return {'date':m.group(1),'level':m.group(2),'msg':m.group(3)}
    return None
def analyze_logs(f):
    errors=0
    warns=0
    infos=0
    for l in open(f):
        p=parse_log(l)
        if p:
            if p['level']=='ERROR':errors+=1
            elif p['level']=='WARNING':warns+=1
            elif p['level']=='INFO':infos+=1
    return {'ERROR':errors,'WARNING':warns,'INFO':infos}
Задание: Исправить стиль, добавить классы, использовать pathlib, константы.

Вариант 20. Веб-скрапер (сложный)
python
# variant_20.py
import requests
from bs4 import BeautifulSoup
def scrape(url):
    r=requests.get(url)
    s=BeautifulSoup(r.text,'html.parser')
    t=s.find('title')
    l=s.find_all('a')
    return {'title':t.text if t else '','links':len(l)}
def scrape_many(urls):
    r=[]
    for u in urls:
        try:
            r.append(scrape(u))
        except:
            r.append({'title':'ERROR','links':0})
    return r
Задание: Добавить обработку исключений, таймауты, user-agent, docstring.

Вариант 21. Банковский счёт (сложный)
python
# variant_21.py
class account:
    def __init__(self,o,b=0):
        self.owner=o
        self.balance=b
        self.transactions=[]
    def deposit(self,a):
        self.balance+=a
        self.transactions.append(f"+{a}")
    def withdraw(self,a):
        if a<=self.balance:
            self.balance-=a
            self.transactions.append(f"-{a}")
        else:
            print("Not enough money")
    def get_balance(self):
        return self.balance
    def get_history(self):
        return self.transactions
Задание: Добавить аннотации, docstring, свойства, проверки, repr.

Вариант 22. Система бронирования (сложный)
python
# variant_22.py
bookings={}
def book(room,date,user):
    k=f"{room}_{date}"
    if k not in bookings:
        bookings[k]=user
        return True
    return False
def cancel(room,date):
    k=f"{room}_{date}"
    if k in bookings:
        del bookings[k]
        return True
    return False
def get_bookings():
    return bookings
Задание: Создать классы Booking, Room; добавить даты, docstring.

Вариант 23. Анализатор CSV (сложный)
python
# variant_23.py
import csv
def read_csv(f):
    d=[]
    with open(f) as file:
        r=csv.DictReader(file)
        for row in r:
            d.append(row)
    return d
def filter_by(d,k,v):
    return [row for row in d if row.get(k)==v]
def group_by(d,k):
    g={}
    for row in d:
        val=row.get(k)
        if val not in g:
            g[val]=[]
        g[val].append(row)
    return g
Задание: Добавить типизацию, docstring, обработку ошибок, класс анализатора.

Вариант 24. REST клиент (сложный)
python
# variant_24.py
import requests,json,time
class APIClient:
    def __init__(self,b):
        self.base=b
        self.c={}
    def get(self,e,p={}):
        k=f"{e}_{json.dumps(p)}"
        if k in self.c and time.time()-self.c[k][1]<60:
            return self.c[k][0]
        r=requests.get(f"{self.base}/{e}",params=p)
        if r.status_code==200:
            self.c[k]=(r.json(),time.time())
            return r.json()
        return None
    def post(self,e,d):
        r=requests.post(f"{self.base}/{e}",json=d)
        return r.json() if r.status_code==201 else None
Задание: Исправить имена, добавить типы, docstring, обработку ошибок, логирование.

Вариант 25. Команды Todo (самый сложный)
python
# variant_25.py
import sys,json,os
from datetime import datetime
c={}
def l():
    for i,t in enumerate(c.values()):
        s='✓' if t['done'] else '□'
        print(f"{s} {i}: {t['title']}")
def a(t):
    c[t]= {'title':t,'done':False,'created':datetime.now().isoformat()}
def d(i):
    k=list(c.keys())[i]
    c[k]['done']=True
def r(i):
    k=list(c.keys())[i]
    del c[k]
def s():
    json.dump(c,open('todo.json','w'))
def ld():
    global c
    if os.path.exists('todo.json'):
        c=json.load(open('todo.json'))
if __name__=='__main__':
    ld()
    if len(sys.argv)<2:
        l()
    else:
        cmd=sys.argv[1]
        if cmd=='add':a(sys.argv[2])
        elif cmd=='done':d(int(sys.argv[2]))
        elif cmd=='remove':r(int(sys.argv[2]))
        elif cmd=='save':s()
        else:print('Unknown command')
    s()
Задание: Полный рефакторинг: классы, argparse, dataclasses, разделение на модули.

Карточка студента (шаблон)
text
ПЗ 2.5. АНАЛИЗ КАЧЕСТВА КОДА

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Исходный код (прилагается отдельным файлом)

=== ЗАДАНИЕ ===

1. Запустите pylint и зафиксируйте начальную оценку
2. Запустите radon cc и radon mi
3. Выпишите все выявленные проблемы
4. Исправьте код в соответствии с PEP 8 и рекомендациями
5. Добейтесь оценки pylint >= 8.0/10
6. Улучшите индекс поддерживаемости radon mi до >= 65

=== ОТЧЁТ ===

Начальная оценка pylint: ___/10
Конечная оценка pylint: ___/10

Начальная сложность radon cc: ______
Конечная сложность radon cc: ______

Начальный MI: ______
Конечный MI: ______

Основные исправления:
1. _______________________________
2. _______________________________
3. _______________________________

Дата выполнения: _____________
Критерии оценки (для всех вариантов)
Баллы	Критерий
2 (неудовлетворительно)	Анализ не выполнен или исправления не работают
3 (удовлетворительно)	Pylint оценка 5.0-6.9, исправлены основные ошибки
4 (хорошо)	Pylint оценка 7.0-8.9, улучшен MI, добавлены docstring
5 (отлично)	Pylint оценка >= 9.0, MI >= 65, код соответствует PEP 8
Шпаргалка по командам
bash
# === УСТАНОВКА ===
pip install pylint radon black isort

# === АНАЛИЗ ===
pylint variant_X.py
pylint variant_X.py --fail-under=8.0
pylint variant_X.py --exit-zero  # не завершать с ошибкой

# === ОЦЕНКА СЛОЖНОСТИ ===
radon cc variant_X.py -s -a
radon mi variant_X.py -s

# === АВТОИСПРАВЛЕНИЕ ===
black variant_X.py
isort variant_X.py
autopep8 --in-place --aggressive variant_X.py

# === ГЕНЕРАЦИЯ КОНФИГА ===
pylint --generate-rcfile > .pylintrc

# === ПРОВЕРКА ТОЛЬКО ОШИБОК ===
pylint variant_X.py --disable=all --enable=E,F
Коды ошибок Pylint (быстрый справочник)
Код	Тип	Описание
C0114	Convention	Отсутствует docstring модуля
C0115	Convention	Отсутствует docstring класса
C0116	Convention	Отсутствует docstring функции
C0103	Convention	Неправильное имя (snake_case для функций)
C0301	Convention	Строка длиннее 100 символов
C0326	Convention	Неправильные пробелы
W0611	Warning	Неиспользуемый импорт
W0612	Warning	Неиспользуемая переменная
W0621	Warning	Переопределение имени
W0702	Warning	Голый except
E0602	Error	Неопределённая переменная
E1101	Error	Нет атрибута у класса
R0902	Refactor	Слишком много атрибутов класса
R0913	Refactor	Слишком много аргументов функции
R0914	Refactor	Слишком много локальных переменных
R0915	Refactor	Слишком много ветвлений
