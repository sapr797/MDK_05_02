# Контрольная работа №3.2 (по разделам 2.14-2.19)

**Дисциплина:** Основы программирования на Python  
**Курс:** 1 курс, СПО  
**Время выполнения:** 90 минут  
**Максимальный балл:** 100  

---

## Содержание

1. [Структура контрольной работы](#1-структура-контрольной-работы)
2. [Перечень тем для контроля](#2-перечень-тем-для-контроля)
3. [Вариант 0 (демонстрационный)](#3-вариант-0-демонстрационный)
4. [25 вариантов заданий](#4-25-вариантов-заданий)
5. [Критерии оценки](#5-критерии-оценки)
6. [Лист ответов](#6-лист-ответов)

---

## 1. Структура контрольной работы

Каждый вариант контрольной работы состоит из **трёх частей**:

| Часть | Разделы | Тип заданий | Баллы |
|-------|---------|-------------|-------|
| **Часть А** (тестовая) | 2.15-2.19 | 10 вопросов с выбором ответа | 30 |
| **Часть В** (практическая) | 2.15-2.17 | Написание кода (30-40 строк) | 40 |
| **Часть С** (аналитическая) | 2.18-2.19 | Анализ и проектирование ETL | 30 |

---

## 2. Перечень тем для контроля

| Раздел | Тема | Ключевые понятия |
|--------|------|------------------|
| **2.15** | Валидация данных | Pydantic, модели, валидаторы, Field, EmailStr |
| **2.16** | Введение в REST API | HTTP-протокол, методы GET/POST/PUT/DELETE, статус-коды |
| **2.17** | Веб-фреймворки | Flask, FastAPI, сравнение, маршруты |
| **2.18** | Клиентские запросы | Fetch API, GET, POST, PUT, DELETE |
| **2.19** | Разработка HTTP-клиента | Библиотека requests, сессии, заголовки |

---

## 3. Вариант 0 (демонстрационный)

**Назначение:** преподаватель выполняет этот вариант на занятии, показывая структуру ответов.

### Часть А. Тестовые задания (30 баллов)

**Выберите один правильный ответ.**

**А1.** Какой метод Pydantic используется для создания модели из словаря?

```text
А) model.dict()
Б) model.parse_obj()
В) model.json()
Г) model.schema()
Правильный ответ: Б

А2. Какой HTTP-метод используется для получения данных?

text
А) POST
Б) PUT
В) GET
Г) DELETE
Правильный ответ: В

А3. Какой статус-код означает успешное создание ресурса?

text
А) 200 OK
Б) 201 Created
В) 204 No Content
Г) 404 Not Found
Правильный ответ: Б

А4. Какой веб-фреймворк является асинхронным по умолчанию?

text
А) Flask
Б) Django
В) FastAPI
Г) Pyramid
Правильный ответ: В

А5. Какой метод Fetch API используется для отправки JSON данных?

text
А) fetch(url, {method: 'GET'})
Б) fetch(url, {method: 'POST', body: JSON.stringify(data)})
В) fetch(url, {method: 'PUT'})
Г) fetch(url, {method: 'DELETE'})
Правильный ответ: Б

А6. Какой параметр Pydantic Field указывает минимальную длину строки?

text
А) min
Б) min_length
В) min_len
Г) minlength
Правильный ответ: Б

А7. Какой HTTP-метод является идемпотентным?

text
А) POST
Б) PATCH
В) PUT
Г) Все ответы верны
Правильный ответ: В

А8. Какой статус-код означает "Не найдено"?

text
А) 200
Б) 400
В) 404
Г) 500
Правильный ответ: В

А9. Какой метод библиотеки requests выполняет POST запрос?

text
А) requests.get()
Б) requests.post()
В) requests.put()
Г) requests.delete()
Правильный ответ: Б

А10. Что делает декоратор @validator в Pydantic?

text
А) Определяет поле модели
Б) Добавляет пользовательскую валидацию
В) Создаёт JSON схему
Г) Преобразует модель в словарь
Правильный ответ: Б

Часть В. Практическое задание (40 баллов)
Задание: Создайте Pydantic модель для валидации данных о книге и напишите функцию, которая валидирует данные из JSON.

Требования к модели Book:

Поле	Тип	Ограничения
id	int	> 0
title	str	min_length=1, max_length=200
author	str	min_length=2, max_length=100
year	int	1500 ≤ year ≤ 2025
isbn	str	формат XXX-X-XXXX-XXXX-X (регулярное выражение)
price	float	> 0
in_stock	bool	по умолчанию True
Дополнительно: Написать функцию validate_book_data(data: dict), которая:

Валидирует данные с помощью Pydantic

Возвращает словарь с ключами is_valid (bool) и errors (список ошибок)

Эталонное решение:

python
from pydantic import BaseModel, Field, validator, ValidationError
from typing import Dict, Any, List, Optional
import re


class Book(BaseModel):
    """Модель книги с валидацией."""
    
    id: int = Field(..., gt=0, description="ID книги")
    title: str = Field(..., min_length=1, max_length=200, description="Название")
    author: str = Field(..., min_length=2, max_length=100, description="Автор")
    year: int = Field(..., ge=1500, le=2025, description="Год издания")
    isbn: str = Field(..., description="ISBN")
    price: float = Field(..., gt=0, description="Цена")
    in_stock: bool = Field(default=True, description="В наличии")
    
    @validator('isbn')
    def validate_isbn(cls, v: str) -> str:
        """Валидация формата ISBN."""
        pattern = r'^\d{3}-\d-\d{4}-\d{4}-\d$'
        if not re.match(pattern, v):
            raise ValueError('Неверный формат ISBN. Ожидается: XXX-X-XXXX-XXXX-X')
        return v
    
    @validator('title', 'author')
    def not_empty(cls, v: str) -> str:
        """Проверка на пустые строки."""
        if not v or not v.strip():
            raise ValueError('Поле не может быть пустым')
        return v.strip().title()


def validate_book_data(data: Dict[str, Any]) -> Dict[str, Any]:
    """
    Валидация данных книги.
    
    Args:
        data: Словарь с данными книги
    
    Returns:
        Словарь с результатами валидации
    """
    try:
        book = Book(**data)
        return {
            "is_valid": True,
            "validated_data": book.dict(),
            "errors": []
        }
    except ValidationError as e:
        errors = []
        for error in e.errors():
            field = '.'.join(str(loc) for loc in error['loc'])
            errors.append({
                "field": field,
                "message": error['msg']
            })
        return {
            "is_valid": False,
            "validated_data": None,
            "errors": errors
        }


# Пример использования
if __name__ == "__main__":
    valid_data = {
        "id": 1,
        "title": "Python для начинающих",
        "author": "Иван Иванов",
        "year": 2024,
        "isbn": "978-5-17-1234567-8",
        "price": 1500.00
    }
    
    result = validate_book_data(valid_data)
    print(f"Валидно: {result['is_valid']}")
    
    invalid_data = {
        "id": 0,
        "title": "",
        "author": "A",
        "year": 3000,
        "isbn": "invalid",
        "price": -100
    }
    
    result = validate_book_data(invalid_data)
    print(f"Валидно: {result['is_valid']}")
    for error in result['errors']:
        print(f"  {error['field']}: {error['message']}")
Часть С. Аналитическое задание (30 баллов)
Задание: Проанализируйте ETL-конвейер для обработки заказов. Найдите ошибки и предложите улучшения с использованием изученных инструментов.

python
# БАД КОД (требует рефакторинга)
def process_orders(file_path):
    data = []
    with open(file_path, 'r') as f:
        for line in f:
            parts = line.strip().split(',')
            if len(parts) >= 5:
                order = {
                    'id': parts[0],
                    'customer': parts[1],
                    'email': parts[2],
                    'amount': float(parts[3]),
                    'status': parts[4]
                }
                data.append(order)
    
    for order in data:
        if '@' not in order['email']:
            print(f"Invalid email: {order['email']}")
            continue
        if order['amount'] <= 0:
            print(f"Invalid amount: {order['amount']}")
            continue
        order['processed'] = True
    
    with open('output.json', 'w') as f:
        import json
        json.dump(data, f)
    
    return data
Эталонный ответ:

Выявленные проблемы:

Отсутствует валидация данных — ручная проверка размазана по коду

Нет обработки ошибок — при неверных данных программа продолжает работу

Смешение ETL этапов — извлечение, валидация, трансформация и загрузка в одной функции

Нет логирования — сложно отследить, что пошло не так

Жёстко заданы пути — нельзя переиспользовать

Нет типизации — непонятно, какие данные ожидаются

Предлагаемые улучшения:

python
from pydantic import BaseModel, Field, validator, EmailStr, ValidationError
from typing import List, Dict, Any
import json
import logging


class Order(BaseModel):
    """Модель заказа с валидацией."""
    
    id: int = Field(..., gt=0)
    customer: str = Field(..., min_length=2, max_length=100)
    email: EmailStr
    amount: float = Field(..., gt=0)
    status: str = Field(..., regex='^(pending|completed|cancelled)$')


class OrderETLProcessor:
    """ETL-процессор для заказов."""
    
    def __init__(self, input_path: str, output_path: str):
        self.input_path = input_path
        self.output_path = output_path
        self.setup_logging()
        self.stats = {"total": 0, "valid": 0, "invalid": 0}
    
    def setup_logging(self):
        logging.basicConfig(level=logging.INFO)
        self.logger = logging.getLogger(__name__)
    
    def extract(self) -> List[Dict]:
        """Извлечение данных из CSV."""
        data = []
        with open(self.input_path, 'r') as f:
            for line in f:
                parts = line.strip().split(',')
                if len(parts) >= 5:
                    data.append({
                        'id': int(parts[0]),
                        'customer': parts[1],
                        'email': parts[2],
                        'amount': float(parts[3]),
                        'status': parts[4]
                    })
        return data
    
    def validate(self, data: List[Dict]) -> List[Order]:
        """Валидация данных."""
        valid_orders = []
        for row in data:
            try:
                order = Order(**row)
                valid_orders.append(order)
                self.stats["valid"] += 1
            except ValidationError as e:
                self.logger.error(f"Validation error: {e}")
                self.stats["invalid"] += 1
            self.stats["total"] += 1
        return valid_orders
    
    def transform(self, orders: List[Order]) -> List[Dict]:
        """Трансформация данных."""
        return [order.dict() for order in orders]
    
    def load(self, data: List[Dict]):
        """Загрузка в JSON."""
        with open(self.output_path, 'w') as f:
            json.dump(data, f, indent=2)
        self.logger.info(f"Loaded {len(data)} orders")
    
    def run(self):
        """Запуск ETL-пайплайна."""
        self.logger.info("Starting ETL process")
        raw_data = self.extract()
        valid_orders = self.validate(raw_data)
        transformed = self.transform(valid_orders)
        self.load(transformed)
        self.logger.info(f"Stats: {self.stats}")


# Использование
processor = OrderETLProcessor("orders.csv", "output.json")
processor.run()
4. 25 вариантов заданий
Варианты 1-8 (Базовый уровень)
Вариант	Часть А (темы)	Часть В (модель)	Часть С (анализ)
1	2.15-2.16	Product	ETL-конвейер для CSV
2	2.16-2.17	User	ETL-конвейер для JSON
3	2.17-2.18	Order	ETL-конвейер для API
4	2.18-2.19	Student	Обработка ошибок
5	2.15-2.17	Employee	Валидация данных
6	2.16-2.18	Movie	Трансформация
7	2.17-2.19	Car	Загрузка
8	2.15-2.19	Laptop	Полный ETL
Варианты 9-17 (Средний уровень)
Вариант	Часть А	Часть В	Часть С
9	2.15-2.16	Book	ETL + DLQ
10	2.16-2.17	Transaction	ETL + логирование
11	2.17-2.18	Customer	ETL + валидация
12	2.18-2.19	Payment	ETL + отчёты
13	2.15-2.17	Invoice	ETL + агрегация
14	2.16-2.18	Delivery	ETL + фильтрация
15	2.17-2.19	Review	ETL + обогащение
16	2.18-2.19	Ticket	ETL + мониторинг
17	2.15-2.19	Hotel	ETL + качество
Варианты 18-25 (Сложный уровень)
Вариант	Часть А	Часть В	Часть С
18	2.15-2.19	Insurance	Полный ETL + DLQ + отчёты
19	2.15-2.19	MedicalRecord	ETL + валидация + логи
20	2.15-2.19	BankTransaction	ETL + алгоритм Луна
21	2.15-2.19	FlightBooking	ETL + интеграционные тесты
22	2.15-2.19	TaxReport	ETL + агрегация + качество
23	2.15-2.19	SocialMedia	ETL + несколько источников
24	2.15-2.19	Ecommerce	Полная система
25	2.15-2.19	Analytics	ETL + дашборд
5. Критерии оценки
Оценка	Баллы	Требования
5 (отлично)	90-100	Часть А: 9-10 правильных ответов. Часть В: модель с валидаторами, функция работает. Часть С: выявлены все проблемы, предложено корректное решение
4 (хорошо)	70-89	Часть А: 7-8 правильных ответов. Часть В: модель создана, есть недочёты. Часть С: проблемы выявлены частично
3 (удовлетворительно)	50-69	Часть А: 5-6 правильных ответов. Часть В: модель с ошибками. Часть С: анализ неполный
2 (неудовлетворительно)	<50	Часть А: менее 5 правильных ответов. Часть В: модель не работает. Часть С: анализ не выполнен
6. Лист ответов (шаблон)
text
ФИО студента: _________________________________
Группа: _________________________________
Вариант №: _________________________________
Дата: _________________________________

=============================================
ЧАСТЬ А. ТЕСТОВЫЕ ЗАДАНИЯ (30 баллов)
=============================================

| Вопрос | А1 | А2 | А3 | А4 | А5 | А6 | А7 | А8 | А9 | А10 |
|--------|----|----|----|----|----|----|----|----|----|-----|
| Ответ  |    |    |    |    |    |    |    |    |    |     |

=============================================
ЧАСТЬ В. ПРАКТИЧЕСКОЕ ЗАДАНИЕ (40 баллов)
=============================================

(Код модели и функции прилагается)

=============================================
ЧАСТЬ С. АНАЛИТИЧЕСКОЕ ЗАДАНИЕ (30 баллов)
=============================================

Выявленные проблемы:
1. _________________________________
2. _________________________________
3. _________________________________

Предложенные улучшения:
1. _________________________________
2. _________________________________
3. _________________________________

=============================================
ИТОГО: _____ / 100 баллов
Оценка: _____
Подпись преподавателя: _____________
=============================================
