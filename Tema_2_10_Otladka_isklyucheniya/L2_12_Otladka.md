# Тема 2.12. Процесс отладки. Точки останова. Обработка исключений: try-except-else-finally

**Цель лекции:**  
Научиться профессионально отлаживать Python-код, использовать точки останова, эффективно обрабатывать ошибки с помощью конструкции `try-except-else-finally`, создавать надежные программы, устойчивые к сбоям.

> Главная мысль: **Ошибки неизбежны. Профессионал отличается от новичка не отсутствием ошибок, а умением их находить и обрабатывать.**

---

## Содержание

1. [Введение в отладку](#1-введение-в-отладку)
2. [Точки останова (Breakpoints)](#2-точки-останова-breakpoints)
3. [Отладка в VS Code](#3-отладка-в-vs-code)
4. [Обработка исключений: try-except](#4-обработка-исключений-try-except)
5. [Конструкция else и finally](#5-конструкция-else-и-finally)
6. [Типы исключений](#6-типы-исключений)
7. [Практические примеры](#7-практические-примеры)
8. [Контрольные вопросы](#8-контрольные-вопросы)
9. [Практическое задание](#9-практическое-задание)
10. [Шпаргалка](#10-шпаргалка)

---

## 1. Введение в отладку

### 1.1. Что такое отладка (debugging)

**Отладка** — это процесс поиска и исправления ошибок в программе.

**Типы ошибок:**

| Тип ошибки | Описание | Пример |
|------------|----------|--------|
| **Синтаксические** | Нарушение правил языка | `print("Hello"` (пропущена скобка) |
| **Логические** | Программа работает, но неверно | `area = width + height` (вместо `*`) |
| **Исключения** | Ошибки во время выполнения | `division by zero`, `file not found` |

### 1.2. Подходы к отладке

| Подход | Описание | Когда использовать |
|--------|----------|-------------------|
| `print()` | Вывод значений переменных | Простые скрипты, быстрая проверка |
| **Точки останова** | Остановка выполнения в IDE | Сложные программы, много переменных |
| **Логирование** | Запись в файл | Программы, работающие долгое время |
| **Тестирование** | Автоматическая проверка | Крупные проекты |

---

## 2. Точки останова (Breakpoints)

### 2.1. Что такое точка останова

**Точка останова (breakpoint)** — это специальная метка в коде, при достижении которой выполнение программы приостанавливается.

### 2.2. Типы точек останова

| Тип | Описание |
|-----|----------|
| **Обычная** | Останавливается всегда |
| **Условная** | Останавливается только при выполнении условия |
| **Счётная** | Останавливается после N проходов |
| **Лог-точка** | Выводит сообщение без остановки |

### 2.3. Условная точка останова

```python
# Пример: остановиться, когда i == 5
for i in range(10):
    result = i * 2      # Точка останова с условием i == 5
    print(result)
В VS Code: Правый клик по точке останова → "Edit Breakpoint" → ввести условие i == 5

2.4. Программные точки останова
python
# Встроенная функция breakpoint() (Python 3.7+)
def calculate(a, b):
    breakpoint()  # Здесь программа остановится
    return a / b

calculate(10, 2)
python
# Условный breakpoint
import pdb

for i in range(10):
    if i == 5:
        pdb.set_trace()  # Остановиться при i == 5
    print(i)
3. Отладка в VS Code
3.1. Основные команды отладчика
Команда	Горячая клавиша	Действие
Start Debugging	F5	Начать отладку
Step Over	F10	Выполнить строку (не заходя в функции)
Step Into	F11	Войти внутрь функции
Step Out	Shift+F11	Выйти из текущей функции
Continue	F5 (в отладке)	Продолжить до следующей точки
Stop	Shift+F5	Остановить отладку
Restart	Ctrl+Shift+F5	Перезапустить
3.2. Настройка отладчика
Файл .vscode/launch.json:

json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "justMyCode": false,
            "args": ["--input", "data.txt"],
            "env": {
                "DEBUG": "true"
            }
        }
    ]
}
3.3. Панель отладки
text
┌─────────────────────────────────────────────────────────┐
│  ВЫПОЛНЕНИЕ (Call Stack)                                 │
│  ├── calculate()  line 15                               │
│  └── main()        line 25                               │
├─────────────────────────────────────────────────────────┤
│  ПЕРЕМЕННЫЕ (Variables)                                  │
│  ├── Локальные                                           │
│  │   ├── a: 10                                           │
│  │   └── b: 0                                            │
│  └── Глобальные                                          │
│      └── __name__: "__main__"                            │
├─────────────────────────────────────────────────────────┤
│  НАБЛЮДЕНИЕ (Watch)                                      │
│  ├── a * b: 0                                            │
│  └── result: undefined                                   │
└─────────────────────────────────────────────────────────┘
3.4. Панель наблюдения (Watch)
Добавление выражений для отслеживания:

python
# Примеры выражений для Watch
i * 2
len(my_list)
my_dict['key']
user.name.upper()
4. Обработка исключений: try-except
4.1. Что такое исключение
Исключение (exception) — это событие, возникающее при ошибке во время выполнения программы.

python
# Пример исключения
result = 10 / 0  # ZeroDivisionError

number = int("abc")  # ValueError

file = open("not_exists.txt")  # FileNotFoundError
4.2. Базовый синтаксис try-except
python
try:
    # Код, который может вызвать ошибку
    result = 10 / 0
    print(result)
except:
    # Код, который выполнится при ошибке
    print("Произошла ошибка!")
4.3. Обработка конкретных исключений
python
try:
    a = int(input("Введите число: "))
    b = int(input("Введите второе число: "))
    result = a / b
    print(f"Результат: {result}")
    
except ValueError:
    print("Ошибка: нужно ввести целое число!")
    
except ZeroDivisionError:
    print("Ошибка: деление на ноль!")
    
except Exception as e:
    print(f"Неизвестная ошибка: {e}")
4.4. Несколько исключений в одном except
python
try:
    value = int(input("Введите число: "))
    result = 100 / value
    
except (ValueError, ZeroDivisionError) as e:
    print(f"Ошибка ввода или деления: {e}")
4.5. Получение информации об ошибке
python
try:
    with open("config.txt", "r") as f:
        data = f.read()
        
except FileNotFoundError as e:
    print(f"Файл не найден: {e.errno}")
    print(f"Текст ошибки: {e.strerror}")
    print(f"Имя файла: {e.filename}")
    
except PermissionError as e:
    print(f"Нет прав доступа: {e}")
4.6. Вложенные try-except
python
try:
    file = open("data.txt", "r")
    try:
        content = file.read()
        number = int(content)
    except ValueError:
        print("Ошибка преобразования данных")
    finally:
        file.close()
except FileNotFoundError:
    print("Файл не найден")
5. Конструкция else и finally
5.1. Блок else
else выполняется, если исключение не было вызвано.

python
try:
    a = int(input("Введите число: "))
    b = int(input("Введите делитель: "))
    result = a / b
    
except ValueError:
    print("Ошибка: введите целые числа!")
    
except ZeroDivisionError:
    print("Ошибка: деление на ноль!")
    
else:
    # Выполнится только если ошибок не было
    print(f"Результат деления: {result}")
    print("Операция выполнена успешно!")
5.2. Блок finally
finally выполняется всегда, независимо от того, была ошибка или нет.

python
try:
    file = open("data.txt", "r")
    data = file.read()
    number = int(data)
    
except FileNotFoundError:
    print("Файл не найден")
    
except ValueError:
    print("Некорректные данные в файле")
    
else:
    print(f"Прочитано число: {number}")
    
finally:
    # Закрываем файл в любом случае
    try:
        file.close()
        print("Файл закрыт")
    except:
        pass
5.3. Полная конструкция
python
try:
    # Код, который может вызвать ошибку
    risky_code()
    
except TypeError:
    # Обработка конкретной ошибки
    handle_type_error()
    
except ValueError as e:
    # Обработка с получением данных об ошибке
    handle_value_error(e)
    
except (ZeroDivisionError, OverflowError):
    # Обработка нескольких ошибок
    handle_math_errors()
    
except Exception as e:
    # Обработка любой другой ошибки
    print(f"Неожиданная ошибка: {e}")
    
else:
    # Выполняется, если ошибок не было
    print("Всё прошло успешно!")
    
finally:
    # Выполняется ВСЕГДА
    cleanup_resources()
5.4. Практический пример: работа с файлом
python
def read_config(filename):
    """Безопасное чтение конфигурационного файла."""
    config = {}
    
    try:
        with open(filename, 'r', encoding='utf-8') as f:
            for line in f:
                if '=' in line:
                    key, value = line.strip().split('=', 1)
                    config[key] = value
                    
    except FileNotFoundError:
        print(f"Файл {filename} не найден. Использую настройки по умолчанию")
        
    except PermissionError:
        print(f"Нет прав на чтение файла {filename}")
        
    except Exception as e:
        print(f"Ошибка при чтении файла: {e}")
        
    else:
        print(f"Файл {filename} успешно загружен")
        
    finally:
        print("Загрузка конфигурации завершена")
    
    return config
6. Типы исключений
6.1. Встроенные исключения Python
Исключение	Описание
SyntaxError	Синтаксическая ошибка (не ловится!)
IndentationError	Ошибка отступов
NameError	Переменная не определена
TypeError	Неверный тип данных
ValueError	Неверное значение
ZeroDivisionError	Деление на ноль
FileNotFoundError	Файл не найден
IndexError	Индекс вне диапазона
KeyError	Ключ не найден в словаре
AttributeError	Атрибут не существует
ImportError	Не удалось импортировать модуль
TimeoutError	Превышено время ожидания
ConnectionError	Ошибка соединения
6.2. Иерархия исключений
text
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── StopIteration
    ├── ArithmeticError
    │   ├── FloatingPointError
    │   ├── OverflowError
    │   └── ZeroDivisionError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── OSError
    │   ├── FileNotFoundError
    │   ├── PermissionError
    │   └── TimeoutError
    ├── TypeError
    ├── ValueError
    └── ...
6.3. Создание своих исключений
python
# Создание пользовательского исключения
class ValidationError(Exception):
    """Исключение для ошибок валидации."""
    pass


class AgeTooLowError(ValidationError):
    """Возраст ниже допустимого."""
    pass


class AgeTooHighError(ValidationError):
    """Возраст выше допустимого."""
    pass


def validate_age(age):
    if age < 0:
        raise ValidationError("Возраст не может быть отрицательным")
    elif age < 18:
        raise AgeTooLowError("Доступ только с 18 лет")
    elif age > 120:
        raise AgeTooHighError("Некорректный возраст")
    return True


# Использование
try:
    validate_age(15)
except AgeTooLowError as e:
    print(f"Ошибка: {e}")
except ValidationError as e:
    print(f"Ошибка валидации: {e}")
7. Практические примеры
7.1. Калькулятор с обработкой ошибок
python
def safe_calculator():
    """Калькулятор с полной обработкой ошибок."""
    print("=" * 40)
    print("    БЕЗОПАСНЫЙ КАЛЬКУЛЯТОР")
    print("=" * 40)
    
    while True:
        try:
            # Ввод первого числа
            a_str = input("\nВведите первое число (или 'exit'): ")
            if a_str.lower() == 'exit':
                print("До свидания!")
                break
            
            a = float(a_str)
            
            # Ввод оператора
            op = input("Введите оператор (+, -, *, /): ")
            
            # Ввод второго числа
            b = float(input("Введите второе число: "))
            
            # Вычисление
            if op == '+':
                result = a + b
            elif op == '-':
                result = a - b
            elif op == '*':
                result = a * b
            elif op == '/':
                if b == 0:
                    raise ZeroDivisionError("Деление на ноль!")
                result = a / b
            else:
                raise ValueError(f"Неизвестный оператор: {op}")
            
            print(f"{a} {op} {b} = {result}")
            
        except ValueError as e:
            print(f"Ошибка ввода: {e}")
            
        except ZeroDivisionError as e:
            print(f"Математическая ошибка: {e}")
            
        except KeyboardInterrupt:
            print("\nПрограмма прервана пользователем")
            break
            
        except Exception as e:
            print(f"Неожиданная ошибка: {e}")
            
        else:
            print("✓ Вычисление выполнено успешно!")
            
        finally:
            print("-" * 40)


if __name__ == "__main__":
    safe_calculator()
7.2. Загрузка данных с повторными попытками
python
import time
import random


def fetch_data_with_retry(url, max_retries=3, delay=1):
    """
    Загружает данные с повторными попытками при ошибке.
    
    Args:
        url: Адрес для запроса
        max_retries: Максимальное количество попыток
        delay: Задержка между попытками (сек)
    
    Returns:
        Данные или None
    """
    import requests
    
    for attempt in range(1, max_retries + 1):
        try:
            print(f"Попытка {attempt}/{max_retries}...")
            response = requests.get(url, timeout=5)
            response.raise_for_status()  # Вызывает исключение при HTTP ошибке
            return response.json()
            
        except requests.exceptions.Timeout:
            print(f"Таймаут при попытке {attempt}")
            
        except requests.exceptions.ConnectionError:
            print(f"Ошибка соединения при попытке {attempt}")
            
        except requests.exceptions.HTTPError as e:
            print(f"HTTP ошибка: {e}")
            if response.status_code == 404:
                break  # Нет смысла повторять
                
        except Exception as e:
            print(f"Неожиданная ошибка: {e}")
            
        if attempt < max_retries:
            wait_time = delay * (2 ** (attempt - 1))  # экспоненциальная задержка
            print(f"Повтор через {wait_time} сек...")
            time.sleep(wait_time)
    
    print("Не удалось загрузить данные после всех попыток")
    return None
7.3. Валидация с подробными ошибками
python
class UserDataValidator:
    """Валидатор пользовательских данных."""
    
    def __init__(self):
        self.errors = []
    
    def validate_email(self, email):
        if not email:
            self.errors.append("Email не может быть пустым")
        elif '@' not in email:
            self.errors.append("Email должен содержать @")
        elif '.' not in email:
            self.errors.append("Email должен содержать точку")
        return len(self.errors) == 0
    
    def validate_age(self, age):
        try:
            age_int = int(age)
            if age_int < 0:
                self.errors.append("Возраст не может быть отрицательным")
            elif age_int > 150:
                self.errors.append("Возраст не может быть больше 150")
            elif age_int < 18:
                self.errors.append("Доступ только с 18 лет")
            return True
        except ValueError:
            self.errors.append("Возраст должен быть числом")
            return False
    
    def validate_phone(self, phone):
        import re
        pattern = r'^\+?[\d\-\(\)\s]{10,}$'
        if not phone:
            self.errors.append("Телефон не может быть пустым")
        elif not re.match(pattern, phone):
            self.errors.append("Неверный формат телефона")
        return len(self.errors) == 0
    
    def validate_all(self, data):
        self.errors = []
        self.validate_email(data.get('email', ''))
        self.validate_age(data.get('age', ''))
        self.validate_phone(data.get('phone', ''))
        return self.errors


# Использование
validator = UserDataValidator()
user_data = {
    'email': 'invalid',
    'age': 'abc',
    'phone': '123'
}

errors = validator.validate_all(user_data)
if errors:
    print("Ошибки валидации:")
    for i, error in enumerate(errors, 1):
        print(f"  {i}. {error}")
else:
    print("Данные корректны!")
8. Контрольные вопросы
Что такое точка останова и зачем она нужна?

Какие типы точек останова существуют? Как создать условную точку останова?

Чем отличается except: от except Exception as e:?

Когда выполняется блок else в конструкции try-except-else?

Зачем нужен блок finally, если он выполняется всегда?

Как создать своё исключение в Python?

В чём разница между ValueError и TypeError?

Как получить подробную информацию об ошибке в блоке except?

Что делает команда breakpoint() в Python 3.7+?

Как настроить отладку в VS Code для Python-проекта?

9. Практическое задание
Задание 1 (базовое)
Напишите функцию safe_divide(a, b), которая возвращает результат деления, а при ошибке возвращает None и выводит сообщение об ошибке.

Задание 2 (среднее)
Напишите программу для чтения чисел из файла. Файл может содержать некорректные данные. Программа должна:

Пропускать некорректные строки

Считать сумму всех корректных чисел

Записывать ошибки в отдельный лог-файл

Использовать try-except-else-finally

Задание 3 (сложное)
Разработайте класс FileProcessor с методами:

read_file(filename) — чтение с обработкой FileNotFoundError, PermissionError

write_file(filename, data) — запись с обработкой ошибок

process_data(data) — обработка с собственным исключением DataError

Использовать with open(...) as file и обработку ошибок

10. Шпаргалка
python
# === БАЗОВАЯ ОБРАБОТКА ===
try:
    risky_code()
except ValueError:
    handle_error()

# === С КОНКРЕТНЫМИ ИСКЛЮЧЕНИЯМИ ===
try:
    value = int(input())
except ValueError as e:
    print(f"Ошибка: {e}")

# === С ELSE И FINALLY ===
try:
    file = open("data.txt")
except FileNotFoundError:
    print("Файл не найден")
else:
    content = file.read()
finally:
    file.close()

# === НЕСКОЛЬКО ИСКЛЮЧЕНИЙ ===
try:
    result = 10 / int(input())
except (ValueError, ZeroDivisionError):
    print("Ошибка ввода или деления")

# === СОЗДАНИЕ СВОЕГО ИСКЛЮЧЕНИЯ ===
class MyError(Exception):
    pass

raise MyError("Сообщение об ошибке")

# === ПРОГРАММНАЯ ТОЧКА ОСТАНОВА ===
breakpoint()  # Python 3.7+

# === ОТЛАДКА В КОНСОЛИ ===
import pdb
pdb.set_trace()

# === УСЛОВНАЯ ТОЧКА ОСТАНОВА ===
for i in range(100):
    if i == 50:
        breakpoint()
    process(i)
Итог лекции
Вы сегодня:

✅ Изучили процесс отладки и типы точек останова

✅ Научились использовать отладчик в VS Code

✅ Освоили конструкцию try-except-else-finally

✅ Узнали о различных типах исключений

✅ Научились создавать свои исключения

✅ Разобрали практические примеры

Теперь ваши программы стали надёжнее, а процесс поиска ошибок — быстрее!
