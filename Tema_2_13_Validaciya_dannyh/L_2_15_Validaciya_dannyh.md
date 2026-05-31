# Тема 2.15. Валидация данных. Библиотека pydantic. Модели, валидаторы

**Цель лекции:**  
Изучить принципы валидации данных в Python, освоить библиотеку Pydantic для создания моделей данных, описания схем, валидации типов и значений.

> Главная мысль: **Никогда не доверяйте внешним данным. Валидация — это защита от ошибок, которых можно было избежать.**

---

## Содержание

1. [Введение в валидацию данных](#1-введение-в-валидацию-данных)
2. [Библиотека Pydantic: основы](#2-библиотека-pydantic-основы)
3. [Модели Pydantic](#3-модели-pydantic)
4. [Валидаторы](#4-валидаторы)
5. [Практические примеры](#5-практические-примеры)
6. [Контрольные вопросы](#6-контрольные-вопросы)
7. [Практическое задание](#7-практическое-задание)
8. [Шпаргалка](#8-шпаргалка)

---

## 1. Введение в валидацию данных

### 1.1. Что такое валидация данных

**Валидация данных** — это процесс проверки данных на соответствие заданным правилам и форматам.

```python
# Пример: что может пойти не так

user_input = input("Введите возраст: ")  # Пользователь может ввести "двадцать"
age = int(user_input)  # ValueError!

user_email = input("Введите email: ")    # "ivan" (без @)
is_valid = '@' in user_email and '.' in user_email  # False
1.2. Типы валидации
Тип валидации	Что проверяет	Пример
Тип данных	Соответствие типу	age: int, name: str
Формат	Соответствие шаблону	Email, телефон, дата
Диапазон	Значение в границах	0 < age < 120
Обязательность	Наличие поля	name не может быть пустым
Уникальность	Отсутствие дубликатов	ID, email
Связность	Зависимость между полями	Если age < 18, то is_adult = False
Бизнес-правила	Специфичные правила	salary >= min_wage
1.3. Подходы к валидации
python
# Плохо: валидация размазана по коду
def create_user(name, email, age):
    if not name:
        raise ValueError("Name required")
    if '@' not in email:
        raise ValueError("Invalid email")
    if age < 0 or age > 150:
        raise ValueError("Invalid age")
    # создание пользователя...

# Лучше: централизованная валидация с Pydantic
from pydantic import BaseModel, EmailStr, Field, validator

class User(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)

# Валидация происходит автоматически при создании
user = User(name="Иван", email="ivan@example.com", age=25)
2. Библиотека Pydantic: основы
2.1. Что такое Pydantic
Pydantic — это библиотека для валидации данных и управления настройками с использованием аннотаций типов Python.

bash
# Установка
pip install pydantic

# Для работы с email и URL нужны дополнительные зависимости
pip install pydantic[email]
2.2. Базовые типы Pydantic
Тип Pydantic	Описание	Пример
str	Строка	name: str
int	Целое число	age: int
float	Число с плавающей точкой	price: float
bool	Логическое значение	is_active: bool
EmailStr	Email (валидация формата)	email: EmailStr
UrlStr	URL (валидация формата)	website: UrlStr
UUID	UUID	id: UUID
datetime	Дата и время	created_at: datetime
date	Дата	birth_date: date
List[T]	Список элементов типа T	tags: List[str]
Dict[K, V]	Словарь	metadata: Dict[str, Any]
Optional[T]	Может быть None	middle_name: Optional[str]
2.3. Простейшая модель
python
from pydantic import BaseModel

class Product(BaseModel):
    id: int
    name: str
    price: float
    in_stock: bool = True  # значение по умолчанию

# Создание экземпляра
product = Product(id=1, name="Ноутбук", price=50000.0)
print(product)
# id=1 name='Ноутбук' price=50000.0 in_stock=True

# Доступ к атрибутам
print(product.name)   # Ноутбук
print(product.price)  # 50000.0

# Преобразование в словарь
print(product.dict())
# {'id': 1, 'name': 'Ноутбук', 'price': 50000.0, 'in_stock': True}

# Преобразование в JSON
print(product.json())
# {"id": 1, "name": "Ноутбук", "price": 50000.0, "in_stock": true}
2.4. Валидация при создании
python
from pydantic import BaseModel, ValidationError

class User(BaseModel):
    name: str
    age: int

# Корректные данные
try:
    user = User(name="Иван", age=25)
    print(f"✅ Пользователь создан: {user}")
except ValidationError as e:
    print(f"Ошибка: {e}")

# Некорректные данные
try:
    user = User(name="Иван", age="двадцать")  # age должен быть int
except ValidationError as e:
    print(f"❌ Ошибка: {e.json()}")
2.5. Field для детальной настройки
python
from pydantic import BaseModel, Field
from typing import Optional

class Article(BaseModel):
    id: int = Field(..., ge=1, description="Уникальный идентификатор")
    title: str = Field(..., min_length=3, max_length=100)
    content: str = Field(..., min_length=10)
    author: str = Field(default="Аноним", max_length=50)
    views: int = Field(default=0, ge=0)
    rating: float = Field(default=0.0, ge=0.0, le=5.0)
    tags: Optional[list] = Field(default_factory=list)

# Параметры Field:
# ... — обязательное поле
# default — значение по умолчанию
# default_factory — функция для создания значения по умолчанию
# gt — больше чем (>)
# ge — больше или равно (>=)
# lt — меньше чем (<)
# le — меньше или равно (<=)
# min_length — минимальная длина строки
# max_length — максимальная длина строки
# regex — регулярное выражение
3. Модели Pydantic
3.1. Вложенные модели
python
from pydantic import BaseModel
from typing import List

class Address(BaseModel):
    city: str
    street: str
    house: int
    apartment: Optional[int] = None

class User(BaseModel):
    name: str
    email: str
    addresses: List[Address]

# Использование
user = User(
    name="Иван",
    email="ivan@example.com",
    addresses=[
        {"city": "Москва", "street": "Тверская", "house": 10},
        {"city": "СПб", "street": "Невский", "house": 25, "apartment": 42}
    ]
)

print(user.json(indent=2))
3.2. Модели с наследованием
python
from pydantic import BaseModel
from datetime import date

class Person(BaseModel):
    name: str
    birth_date: date

    @property
    def age(self) -> int:
        today = date.today()
        return today.year - self.birth_date.year

class Employee(Person):
    employee_id: str
    department: str
    salary: float

class Manager(Employee):
    subordinates: list = []

# Использование
manager = Manager(
    name="Анна",
    birth_date="1985-06-15",
    employee_id="E001",
    department="IT",
    salary=150000,
    subordinates=["E002", "E003"]
)

print(f"Менеджер: {manager.name}")
print(f"Возраст: {manager.age}")
3.3. Generic модели (типизированные)
python
from pydantic import BaseModel
from typing import Generic, TypeVar, List

T = TypeVar('T')

class PaginatedResponse(BaseModel, Generic[T]):
    items: List[T]
    total: int
    page: int
    page_size: int

    @property
    def total_pages(self) -> int:
        return (self.total + self.page_size - 1) // self.page_size

class Product(BaseModel):
    id: int
    name: str
    price: float

class User(BaseModel):
    id: int
    name: str
    email: str

# Специализация для разных типов
product_page = PaginatedResponse[Product](
    items=[Product(id=1, name="Ноутбук", price=50000)],
    total=10,
    page=1,
    page_size=20
)

user_page = PaginatedResponse[User](
    items=[User(id=1, name="Иван", email="ivan@example.com")],
    total=5,
    page=1,
    page_size=20
)
3.4. Методы модели
python
from pydantic import BaseModel
from datetime import datetime

class Post(BaseModel):
    id: int
    title: str
    content: str
    created_at: datetime = None

    def __init__(self, **data):
        if 'created_at' not in data:
            data['created_at'] = datetime.now()
        super().__init__(**data)
    
    def summary(self, length: int = 50) -> str:
        return self.content[:length] + ("..." if len(self.content) > length else "")
    
    def dict(self, **kwargs):
        data = super().dict(**kwargs)
        data['summary'] = self.summary()
        return data

# Использование
post = Post(id=1, title="Новость", content="Очень длинное содержание новости...")
print(post.summary())
print(post.dict())
3.5. Парсинг данных
python
from pydantic import BaseModel, parse_obj_as
from typing import List

class Product(BaseModel):
    id: int
    name: str
    price: float

# Из словаря
product = Product.parse_obj({"id": 1, "name": "Ноутбук", "price": 50000})

# Из JSON строки
product_json = '{"id": 2, "name": "Мышь", "price": 1500}'
product = Product.parse_raw(product_json)

# Список продуктов
products_data = [
    {"id": 1, "name": "Ноутбук", "price": 50000},
    {"id": 2, "name": "Мышь", "price": 1500}
]
products = parse_obj_as(List[Product], products_data)

# Из файла
with open('products.json', 'r') as f:
    products = Product.parse_file('products.json')
4. Валидаторы
4.1. Базовый валидатор (@validator)
python
from pydantic import BaseModel, validator
import re

class User(BaseModel):
    name: str
    email: str
    age: int
    
    @validator('name')
    def name_must_be_valid(cls, v):
        if not v.strip():
            raise ValueError('Имя не может быть пустым')
        if len(v) < 2:
            raise ValueError('Имя должно содержать минимум 2 символа')
        return v.strip().title()
    
    @validator('email')
    def email_must_be_valid(cls, v):
        if '@' not in v or '.' not in v:
            raise ValueError('Неверный формат email')
        return v.lower()
    
    @validator('age')
    def age_must_be_valid(cls, v):
        if v < 0:
            raise ValueError('Возраст не может быть отрицательным')
        if v > 150:
            raise ValueError('Некорректный возраст')
        return v

# Тестирование
try:
    user = User(name="  иван  ", email="IVAN@EXAMPLE.COM", age=25)
    print(f"✅ Пользователь: {user.name}, {user.email}")
except ValueError as e:
    print(f"❌ Ошибка: {e}")
4.2. Валидаторы с использованием регулярных выражений
python
from pydantic import BaseModel, validator
import re

class Contact(BaseModel):
    phone: str
    passport: str
    
    @validator('phone')
    def validate_phone(cls, v):
        pattern = r'^\+?7\d{10}$|^8\d{10}$'
        if not re.match(pattern, v):
            raise ValueError('Неверный формат телефона. Ожидается: +71234567890 или 81234567890')
        # Нормализация: убираем +, заменяем 8 на +7
        if v.startswith('8'):
            v = '+7' + v[1:]
        elif v.startswith('+7'):
            pass
        else:
            v = '+7' + v
        return v
    
    @validator('passport')
    def validate_passport(cls, v):
        pattern = r'^\d{4}\s?\d{6}$'
        if not re.match(pattern, v):
            raise ValueError('Неверный формат паспорта. Ожидается: 1234 567890 или 1234567890')
        # Нормализация: убираем пробел
        return v.replace(' ', '')

contact = Contact(phone="+71234567890", passport="1234567890")
print(contact.phone)  # +71234567890
print(contact.passport)  # 1234567890
4.3. Валидация нескольких полей (@root_validator)
python
from pydantic import BaseModel, root_validator

class Order(BaseModel):
    items: list
    total: float
    discount: float = 0
    final_total: float = None
    
    @root_validator
    def validate_order(cls, values):
        items = values.get('items', [])
        total = values.get('total', 0)
        discount = values.get('discount', 0)
        
        # Проверка: если есть товары, total не должен быть 0
        if items and total == 0:
            raise ValueError('При наличии товаров сумма не может быть 0')
        
        # Проверка: скидка не может превышать 100%
        if discount > total:
            raise ValueError('Скидка не может превышать сумму заказа')
        
        # Вычисление финальной суммы
        values['final_total'] = total - discount
        return values

# Тестирование
order = Order(items=[1, 2, 3], total=1000, discount=150)
print(f"Итого: {order.final_total}")  # 850

try:
    order = Order(items=[1, 2, 3], total=0, discount=0)
except ValueError as e:
    print(f"❌ Ошибка: {e}")
4.4. Предварительные валидаторы (pre=True)
python
from pydantic import BaseModel, validator

class Product(BaseModel):
    price: float
    quantity: int
    
    @validator('price', pre=True)
    def parse_price(cls, v):
        """Преобразует строку с деньгами в число."""
        if isinstance(v, str):
            # Убираем символы валют, заменяем запятую на точку
            v = v.replace('$', '').replace('€', '').replace('₽', '')
            v = v.replace(',', '.')
            return float(v)
        return v
    
    @validator('quantity', pre=True)
    def parse_quantity(cls, v):
        """Преобразует строку в число."""
        if isinstance(v, str):
            return int(v)
        return v

# Тестирование
product = Product(price="$1,299.99", quantity="5")
print(f"Цена: {product.price}, Количество: {product.quantity}")
4.5. Валидаторы для всего поля (each_item)
python
from pydantic import BaseModel, validator
from typing import List

class ShoppingCart(BaseModel):
    prices: List[float]
    
    @validator('prices', each_item=True)
    def price_must_be_positive(cls, v):
        if v < 0:
            raise ValueError('Цена не может быть отрицательной')
        return v
    
    @validator('prices', each_item=True)
    def price_must_be_reasonable(cls, v):
        if v > 1_000_000:
            raise ValueError('Цена не может превышать 1 000 000')
        return v

# Тестирование
cart = ShoppingCart(prices=[100, 250, 3000])
print(f"Корзина: {cart.prices}")

try:
    cart = ShoppingCart(prices=[100, -50, 300])
except ValueError as e:
    print(f"❌ Ошибка: {e}")
5. Практические примеры
5.1. Валидация конфигурации приложения
python
from pydantic import BaseModel, Field, validator
from typing import Optional
import os
from pathlib import Path

class AppConfig(BaseModel):
    app_name: str = Field("MyApp", min_length=1, max_length=50)
    debug: bool = False
    port: int = Field(8000, ge=1024, le=65535)
    host: str = "127.0.0.1"
    log_level: str = Field("INFO", regex="^(DEBUG|INFO|WARNING|ERROR|CRITICAL)$")
    data_dir: Optional[Path] = None
    
    @validator('data_dir', pre=True)
    def validate_data_dir(cls, v):
        if v is None:
            v = Path.cwd() / "data"
        v = Path(v)
        v.mkdir(parents=True, exist_ok=True)
        return v
    
    @classmethod
    def from_env(cls):
        """Загружает конфигурацию из переменных окружения."""
        return cls(
            app_name=os.getenv('APP_NAME', 'MyApp'),
            debug=os.getenv('DEBUG', 'false').lower() == 'true',
            port=int(os.getenv('PORT', '8000')),
            host=os.getenv('HOST', '127.0.0.1'),
            log_level=os.getenv('LOG_LEVEL', 'INFO'),
            data_dir=os.getenv('DATA_DIR', None)
        )

# Использование
config = AppConfig.from_env()
print(f"Приложение: {config.app_name}")
print(f"Порт: {config.port}")
print(f"Папка данных: {config.data_dir}")
5.2. Валидация API запросов
python
from pydantic import BaseModel, Field, validator
from typing import Optional, List
from datetime import datetime
import re

class CreateUserRequest(BaseModel):
    username: str = Field(..., min_length=3, max_length=20)
    email: str
    password: str = Field(..., min_length=8)
    age: Optional[int] = Field(None, ge=0, le=150)
    interests: List[str] = Field(default_factory=list)
    
    @validator('username')
    def username_alphanumeric(cls, v):
        if not re.match(r'^[a-zA-Z0-9_]+$', v):
            raise ValueError('Имя пользователя может содержать только буквы, цифры и подчёркивание')
        return v.lower()
    
    @validator('email')
    def email_valid(cls, v):
        if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', v):
            raise ValueError('Неверный формат email')
        return v.lower()
    
    @validator('password')
    def password_strong(cls, v):
        if not re.search(r'[A-Z]', v):
            raise ValueError('Пароль должен содержать хотя бы одну заглавную букву')
        if not re.search(r'[0-9]', v):
            raise ValueError('Пароль должен содержать хотя бы одну цифру')
        return v
    
    class Config:
        schema_extra = {
            "example": {
                "username": "ivan123",
                "email": "ivan@example.com",
                "password": "SecurePass123",
                "age": 25,
                "interests": ["python", "music"]
            }
        }

# Использование
try:
    user = CreateUserRequest(
        username="ivan123",
        email="ivan@example.com",
        password="SecurePass123",
        age=25
    )
    print(f"✅ Пользователь создан: {user.username}")
except ValueError as e:
    print(f"❌ Ошибка: {e}")
5.3. Валидация ответа от API
python
from pydantic import BaseModel, Field
from typing import List, Optional
from datetime import datetime

class WeatherData(BaseModel):
    temperature: float = Field(..., alias='temp')
    feels_like: float = Field(..., alias='feels_like')
    humidity: int
    pressure: int
    wind_speed: float = Field(..., alias='wind_speed')
    
    class Config:
        allow_population_by_field_name = True

class WeatherResponse(BaseModel):
    city: str
    country: str
    timestamp: datetime
    data: WeatherData
    
    @property
    def description(self) -> str:
        return f"{self.city}, {self.country}: {self.data.temperature}°C"

# Пример ответа от API
api_response = {
    "city": "Moscow",
    "country": "RU",
    "timestamp": "2024-01-15T14:30:00",
    "data": {
        "temp": -5.2,
        "feels_like": -9.1,
        "humidity": 85,
        "pressure": 1012,
        "wind_speed": 4.5
    }
}

weather = WeatherResponse.parse_obj(api_response)
print(weather.description)
5.4. Валидация данных из CSV
python
import csv
from pydantic import BaseModel, validator, ValidationError
from typing import List
from pathlib import Path

class Employee(BaseModel):
    id: int
    name: str
    email: str
    salary: int
    
    @validator('name')
    def name_not_empty(cls, v):
        if not v or not v.strip():
            raise ValueError('Имя не может быть пустым')
        return v.strip()
    
    @validator('email')
    def email_valid(cls, v):
        if '@' not in v or '.' not in v:
            raise ValueError('Неверный формат email')
        return v.lower()
    
    @validator('salary')
    def salary_positive(cls, v):
        if v < 0:
            raise ValueError('Зарплата не может быть отрицательной')
        return v

def load_employees_from_csv(filepath: Path) -> List[Employee]:
    """Загружает и валидирует сотрудников из CSV файла."""
    employees = []
    errors = []
    
    with open(filepath, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        for row_num, row in enumerate(reader, start=2):
            try:
                employee = Employee.parse_obj(row)
                employees.append(employee)
            except ValidationError as e:
                errors.append({
                    'line': row_num,
                    'data': row,
                    'errors': e.errors()
                })
    
    if errors:
        print(f"⚠️ Обнаружено {len(errors)} ошибок валидации:")
        for err in errors[:5]:
            print(f"   Строка {err['line']}: {err['errors']}")
    
    return employees

# Использование
employees = load_employees_from_csv(Path("employees.csv"))
print(f"Загружено сотрудников: {len(employees)}")
6. Контрольные вопросы
Что такое валидация данных и почему она важна?

Какие типы валидации данных вы знаете?

Что такое Pydantic и для чего он используется?

Как создать модель Pydantic с обязательными и необязательными полями?

В чём разница между Field(...) и Field(default=...)?

Как создать валидатор для одного поля? Приведите пример.

Как создать валидатор, который проверяет несколько полей одновременно?

Что делает параметр pre=True в декораторе @validator?

Как работать с вложенными моделями в Pydantic?

Как использовать Pydantic для валидации данных из JSON/CSV?

7. Практическое задание
Задание 1 (базовое)
Создайте модель User с полями: name, email, age, is_active. Добавьте валидацию: имя не пустое, email корректен, возраст от 0 до 120.

Задание 2 (среднее)
Создайте модель Order с полями: order_id, items (список), total. Добавьте валидатор, который проверяет, что total соответствует сумме цен товаров.

Задание 3 (сложное)
Создайте модель Config для валидации конфигурации приложения с поддержкой:

Загрузки из JSON файла

Загрузки из переменных окружения

Валидации путей и прав доступа

8. Шпаргалка
python
# === ОСНОВНЫЕ ИМПОРТЫ ===
from pydantic import BaseModel, Field, validator, root_validator
from pydantic import EmailStr, UrlStr, ValidationError
from typing import List, Dict, Optional

# === БАЗОВАЯ МОДЕЛЬ ===
class Model(BaseModel):
    field: type = Field(..., description="Описание")
    
# === FIELD ПАРАМЕТРЫ ===
# ... - обязательное поле
# default - значение по умолчанию
# gt, ge, lt, le - сравнение
# min_length, max_length - длина строки
# regex - регулярное выражение

# === ВАЛИДАТОР ПОЛЯ ===
@validator('field_name')
def validate_field(cls, v):
    if not condition:
        raise ValueError('Ошибка')
    return v

# === ВАЛИДАТОР НЕСКОЛЬКИХ ПОЛЕЙ ===
@root_validator
def validate_all(cls, values):
    field1 = values.get('field1')
    field2 = values.get('field2')
    if condition:
        raise ValueError('Ошибка')
    return values

# === ПРЕОБРАЗОВАНИЕ ===
# model.dict() - в словарь
# model.json() - в JSON строку
# Model.parse_obj(dict) - из словаря
# Model.parse_raw(json_str) - из JSON строки
# Model.parse_file(filepath) - из файла
Итог лекции
Вы сегодня:

✅ Изучили важность валидации данных

✅ Освоили библиотеку Pydantic для создания моделей данных

✅ Научились использовать Field для детальной настройки полей

✅ Изучили создание валидаторов для одного и нескольких полей

✅ Разобрали практические примеры валидации конфигурации, API запросов и CSV файлов

Теперь ваши приложения будут устойчивы к некорректным данным!

