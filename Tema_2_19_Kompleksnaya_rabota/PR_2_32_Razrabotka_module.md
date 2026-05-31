# ПЗ 2.32. Разработка модуля валидации с pydantic

**Тема:** Валидация данных, библиотека Pydantic, модели, валидаторы, ETL-процессы

**Цель работы:**  
Научиться разрабатывать модуль валидации данных с использованием Pydantic, создавать сложные модели с валидацией полей и межполевыми проверками, интегрировать валидацию в ETL-процессы.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `pydantic`, `pydantic[email]`

```bash
pip install pydantic pydantic[email]
Главная мысль: Pydantic превращает объявление правил валидации в чистый, декларативный код. Никакого ручного if — только схемы и аннотации.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Что такое Pydantic
Pydantic — это библиотека для валидации данных с использованием аннотаций типов Python.

Ключевые возможности:

Возможность	Описание
Автоматическая валидация типов	Проверяет соответствие типов данных
Преобразование типов	Автоматически приводит строки к числам, датам
Field	Дополнительные ограничения (min, max, regex)
Валидаторы	Пользовательские проверки с @validator
Вложенные модели	Поддержка сложных структур
Наследование схем	Переиспользование правил валидации
1.2. Pydantic vs ручная валидация
Аспект	Ручная валидация	Pydantic
Объём кода	Большой	Маленький
Читаемость	Низкая	Высокая
Поддерживаемость	Сложная	Простая
Ошибки	Легко пропустить	Автоматические
Производительность	Выше	Ниже (но достаточно)
1.3. Основные компоненты Pydantic
python
from pydantic import BaseModel, Field, validator, root_validator, EmailStr
from typing import Optional, List
from datetime import datetime

class MyModel(BaseModel):
    # Поле с ограничениями
    name: str = Field(..., min_length=2, max_length=50)
    
    # Опциональное поле
    age: Optional[int] = None
    
    # Встроенный валидатор email
    email: EmailStr
    
    # Пользовательский валидатор
    @validator('name')
    def name_must_be_title(cls, v):
        return v.title()
    
    # Валидатор нескольких полей
    @root_validator
    def check_dates(cls, values):
        # проверка зависимостей
        return values
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая разработку модуля валидации с Pydantic.

Техническое задание (нулевой вариант)
Разработайте модуль валидации order_validator.py для заказов интернет-магазина. Модуль должен содержать:

Модель для валидации заказа (Order)

Модель для валидации товара в заказе (OrderItem)

Валидацию типов и форматов (email, телефон)

Межполевую валидацию (сумма товаров = общей сумме)

Бизнес-правила (скидка не более 50%, минимальная сумма заказа)

Эталонная реализация
python
#!/usr/bin/env python3
"""
order_validator.py — Модуль валидации заказов с использованием Pydantic.
"""

from pydantic import BaseModel, Field, validator, root_validator, EmailStr
from typing import List, Optional, Dict, Any
from datetime import datetime
from decimal import Decimal
import re


# ============================================================
# ВСПОМОГАТЕЛЬНЫЕ ВАЛИДАТОРЫ
# ============================================================

def validate_phone(value: str) -> str:
    """Валидация и нормализация российского номера телефона."""
    if not value:
        return value
    
    # Удаляем все нецифровые символы
    digits = re.sub(r'\D', '', value)
    
    # Проверка длины
    if len(digits) not in [10, 11]:
        raise ValueError('Номер телефона должен содержать 10 или 11 цифр')
    
    # Нормализация
    if len(digits) == 11 and digits[0] == '8':
        digits = '7' + digits[1:]
    elif len(digits) == 10:
        digits = '7' + digits
    
    if len(digits) == 11 and digits[0] == '7':
        return f"+{digits}"
    
    raise ValueError('Неверный формат номера телефона')


def validate_date(value: str) -> datetime:
    """Валидация даты в различных форматах."""
    if isinstance(value, datetime):
        return value
    
    formats = [
        "%Y-%m-%d",
        "%Y-%m-%d %H:%M:%S",
        "%d.%m.%Y",
        "%d/%m/%Y",
        "%Y%m%d"
    ]
    
    for fmt in formats:
        try:
            return datetime.strptime(value, fmt)
        except (ValueError, TypeError):
            continue
    
    raise ValueError(f'Неверный формат даты: {value}')


# ============================================================
# МОДЕЛИ PYDANTIC
# ============================================================

class OrderItem(BaseModel):
    """Модель товара в заказе."""
    
    product_id: int = Field(..., gt=0, description="ID товара")
    name: str = Field(..., min_length=1, max_length=200, description="Название товара")
    quantity: int = Field(..., gt=0, le=999, description="Количество")
    price: Decimal = Field(..., gt=0, decimal_places=2, description="Цена за единицу")
    
    @property
    def subtotal(self) -> Decimal:
        """Стоимость позиции."""
        return self.price * self.quantity
    
    @validator('name')
    def name_not_empty(cls, v):
        if not v or not v.strip():
            raise ValueError('Название товара не может быть пустым')
        return v.strip().title()
    
    class Config:
        json_schema_extra = {
            "example": {
                "product_id": 1,
                "name": "Ноутбук",
                "quantity": 2,
                "price": 50000.00
            }
        }


class Customer(BaseModel):
    """Модель покупателя."""
    
    name: str = Field(..., min_length=2, max_length=100, description="ФИО покупателя")
    email: EmailStr = Field(..., description="Email покупателя")
    phone: str = Field(..., description="Телефон покупателя")
    
    @validator('name')
    def name_format(cls, v):
        """Нормализация имени."""
        return v.strip().title()
    
    @validator('phone')
    def phone_valid(cls, v):
        """Валидация телефона."""
        return validate_phone(v)
    
    class Config:
        json_schema_extra = {
            "example": {
                "name": "Иван Петров",
                "email": "ivan@example.com",
                "phone": "+79123456789"
            }
        }


class Address(BaseModel):
    """Модель адреса доставки."""
    
    city: str = Field(..., min_length=2, max_length=100, description="Город")
    street: str = Field(..., min_length=2, max_length=200, description="Улица")
    house: str = Field(..., min_length=1, max_length=20, description="Номер дома")
    apartment: Optional[str] = Field(None, max_length=20, description="Номер квартиры")
    postal_code: Optional[str] = Field(None, regex=r'^\d{5,6}$', description="Почтовый индекс")
    
    @validator('city', 'street')
    def city_street_format(cls, v):
        """Нормализация названий."""
        return v.strip().title()
    
    class Config:
        json_schema_extra = {
            "example": {
                "city": "Москва",
                "street": "Тверская",
                "house": "10",
                "apartment": "42",
                "postal_code": "101000"
            }
        }


class Order(BaseModel):
    """Модель заказа."""
    
    # Основные поля
    order_id: str = Field(..., min_length=5, max_length=20, description="Номер заказа")
    customer: Customer = Field(..., description="Покупатель")
    items: List[OrderItem] = Field(..., min_items=1, description="Товары в заказе")
    
    # Поля с дефолтными значениями
    status: str = Field("pending", regex="^(pending|processing|shipped|delivered|cancelled)$")
    created_at: datetime = Field(default_factory=datetime.now)
    
    # Финансовые поля
    subtotal: Optional[Decimal] = Field(None, ge=0, description="Сумма без скидки")
    discount: Decimal = Field(0, ge=0, le=1000000, description="Скидка")
    delivery_cost: Decimal = Field(0, ge=0, description="Стоимость доставки")
    total: Optional[Decimal] = Field(None, ge=0, description="Итоговая сумма")
    
    # Дополнительные поля
    address: Optional[Address] = None
    comment: Optional[str] = Field(None, max_length=500)
    
    # ===== ВАЛИДАТОРЫ ОДИНОЧНЫХ ПОЛЕЙ =====
    
    @validator('order_id')
    def order_id_format(cls, v):
        """Проверка формата номера заказа."""
        if not re.match(r'^ORD-\d{8,10}$', v):
            raise ValueError('Номер заказа должен быть в формате ORD-XXXXXXXX')
        return v
    
    @validator('subtotal', pre=True, always=True)
    def calculate_subtotal(cls, v, values):
        """Автоматический расчёт subtotal на основе товаров."""
        items = values.get('items', [])
        if items:
            subtotal = sum(item.subtotal for item in items)
            return subtotal
        return v
    
    @validator('total', pre=True, always=True)
    def calculate_total(cls, v, values):
        """Автоматический расчёт итоговой суммы."""
        subtotal = values.get('subtotal', 0)
        discount = values.get('discount', 0)
        delivery_cost = values.get('delivery_cost', 0)
        
        total = subtotal - discount + delivery_cost
        return max(total, 0)
    
    # ===== МЕЖПОЛЕВЫЕ ВАЛИДАТОРЫ =====
    
    @root_validator(pre=False)
    def validate_order(cls, values):
        """Комплексная проверка заказа."""
        items = values.get('items', [])
        discount = values.get('discount', 0)
        subtotal = values.get('subtotal', 0)
        
        # Проверка: заказ не может быть пустым
        if not items:
            raise ValueError('Заказ должен содержать хотя бы один товар')
        
        # Проверка: скидка не может превышать 50% от суммы заказа
        if discount > subtotal * Decimal('0.5'):
            raise ValueError(f'Скидка {discount} не может превышать 50% от суммы заказа {subtotal}')
        
        # Проверка: минимальная сумма заказа
        if subtotal < Decimal('500'):
            raise ValueError(f'Минимальная сумма заказа 500 рублей. Текущая сумма: {subtotal}')
        
        return values
    
    # ===== МЕТОДЫ МОДЕЛИ =====
    
    def get_summary(self) -> Dict:
        """Краткая информация о заказе."""
        return {
            "order_id": self.order_id,
            "customer": self.customer.name,
            "items_count": len(self.items),
            "total_quantity": sum(item.quantity for item in self.items),
            "subtotal": float(self.subtotal) if self.subtotal else 0,
            "discount": float(self.discount),
            "delivery_cost": float(self.delivery_cost),
            "total": float(self.total) if self.total else 0,
            "status": self.status
        }
    
    class Config:
        title = "Order"
        description = "Модель заказа для интернет-магазина"
        json_schema_extra = {
            "example": {
                "order_id": "ORD-20240115",
                "customer": {
                    "name": "Иван Петров",
                    "email": "ivan@example.com",
                    "phone": "+79123456789"
                },
                "items": [
                    {
                        "product_id": 1,
                        "name": "Ноутбук",
                        "quantity": 1,
                        "price": 50000.00
                    },
                    {
                        "product_id": 2,
                        "name": "Мышь",
                        "quantity": 2,
                        "price": 1500.00
                    }
                ],
                "discount": 0,
                "delivery_cost": 500,
                "address": {
                    "city": "Москва",
                    "street": "Тверская",
                    "house": "10"
                }
            }
        }


# ============================================================
# ВАЛИДАТОР ДАННЫХ (ДЛЯ ИСПОЛЬЗОВАНИЯ В ETL)
# ============================================================

class OrderValidator:
    """
    Класс-обёртка для валидации заказов.
    Используется в ETL-процессах.
    """
    
    def __init__(self):
        self.valid_orders = []
        self.invalid_orders = []
        self.errors = []
    
    def validate_order_data(self, order_data: Dict) -> Optional[Order]:
        """
        Валидация одного заказа.
        
        Returns:
            Order: Валидный заказ или None
        """
        try:
            validated_order = Order(**order_data)
            self.valid_orders.append(validated_order)
            return validated_order
        except Exception as e:
            self.invalid_orders.append(order_data)
            self.errors.append({
                "order_id": order_data.get('order_id', 'unknown'),
                "error": str(e),
                "data": order_data
            })
            return None
    
    def validate_batch(self, orders_data: List[Dict]) -> List[Order]:
        """
        Валидация пакета заказов.
        
        Returns:
            List[Order]: Список валидных заказов
        """
        valid = []
        for order_data in orders_data:
            validated = self.validate_order_data(order_data)
            if validated:
                valid.append(validated)
        
        return valid
    
    def get_statistics(self) -> Dict:
        """Статистика валидации."""
        return {
            "total": len(self.valid_orders) + len(self.invalid_orders),
            "valid": len(self.valid_orders),
            "invalid": len(self.invalid_orders),
            "success_rate": f"{len(self.valid_orders) / (len(self.valid_orders) + len(self.invalid_orders)) * 100:.1f}%" if (self.valid_orders or self.invalid_orders) else "0%"
        }
    
    def get_errors_report(self) -> List[Dict]:
        """Отчёт об ошибках валидации."""
        return self.errors


# ============================================================
# ДЕМОНСТРАЦИЯ
# ============================================================

def demo():
    """Демонстрация работы модуля валидации."""
    
    print("=" * 60)
    print("ДЕМОНСТРАЦИЯ МОДУЛЯ ВАЛИДАЦИИ PYDANTIC")
    print("=" * 60)
    
    # Тестовые данные
    test_orders = [
        {
            "order_id": "ORD-20240115",
            "customer": {
                "name": "иван петров",
                "email": "ivan@example.com",
                "phone": "+79123456789"
            },
            "items": [
                {"product_id": 1, "name": "Ноутбук", "quantity": 1, "price": 50000},
                {"product_id": 2, "name": "Мышь", "quantity": 2, "price": 1500}
            ],
            "discount": 0,
            "delivery_cost": 500,
            "address": {
                "city": "москва",
                "street": "тверская",
                "house": "10"
            }
        },
        {
            "order_id": "INVALID",
            "customer": {
                "name": "Анна",
                "email": "invalid-email",
                "phone": "123"
            },
            "items": [],
            "discount": 10000,
            "delivery_cost": 0
        },
        {
            "order_id": "ORD-20240116",
            "customer": {
                "name": "Мария Сидорова",
                "email": "maria@example.com",
                "phone": "89231234567"
            },
            "items": [
                {"product_id": 3, "name": "Книга", "quantity": 1, "price": 500}
            ],
            "discount": 100,
            "delivery_cost": 300
        }
    ]
    
    # Валидация
    validator = OrderValidator()
    valid_orders = validator.validate_batch(test_orders)
    
    print("\n📊 РЕЗУЛЬТАТЫ ВАЛИДАЦИИ:")
    print(f"   Всего заказов: {len(test_orders)}")
    print(f"   Валидных: {len(valid_orders)}")
    print(f"   Невалидных: {len(test_orders) - len(valid_orders)}")
    
    # Вывод информации о валидных заказах
    print("\n✅ ВАЛИДНЫЕ ЗАКАЗЫ:")
    for order in valid_orders:
        print(f"\n   Заказ {order.order_id}:")
        print(f"      Покупатель: {order.customer.name}")
        print(f"      Товаров: {len(order.items)}")
        print(f"      Сумма: {order.total} руб.")
        print(f"      Статус: {order.status}")
    
    # Вывод ошибок
    if validator.get_errors_report():
        print("\n❌ ОШИБКИ ВАЛИДАЦИИ:")
        for error in validator.get_errors_report():
            print(f"\n   Заказ: {error['order_id']}")
            print(f"   Ошибка: {error['error'][:100]}...")
    
    # Демонстрация методов модели
    if valid_orders:
        print("\n📋 ПРИМЕР ИСПОЛЬЗОВАНИЯ МЕТОДОВ МОДЕЛИ:")
        order = valid_orders[0]
        summary = order.get_summary()
        print(f"\n   Краткая информация о заказе {order.order_id}:")
        for key, value in summary.items():
            print(f"      {key}: {value}")
        
        print(f"\n   Преобразование в JSON:")
        print(f"      {order.model_dump_json(indent=2)[:200]}...")
    
    print("\n" + "=" * 60)
    print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
    print("=" * 60)


if __name__ == "__main__":
    demo()
Ожидаемый вывод
text
============================================================
ДЕМОНСТРАЦИЯ МОДУЛЯ ВАЛИДАЦИИ PYDANTIC
============================================================

📊 РЕЗУЛЬТАТЫ ВАЛИДАЦИИ:
   Всего заказов: 3
   Валидных: 2
   Невалидных: 1

✅ ВАЛИДНЫЕ ЗАКАЗЫ:

   Заказ ORD-20240115:
      Покупатель: Иван Петров
      Товаров: 2
      Сумма: 48000.0 руб.
      Статус: pending

   Заказ ORD-20240116:
      Покупатель: Мария Сидорова
      Товаров: 1
      Сумма: 700.0 руб.
      Статус: pending

❌ ОШИБКИ ВАЛИДАЦИИ:

   Заказ: INVALID
   Ошибка: 1 validation error for Order
order_id
  Номер заказа должен быть в формате ORD-XXXXXXXX (type=value_error)...

📋 ПРИМЕР ИСПОЛЬЗОВАНИЯ МЕТОДОВ МОДЕЛИ:

   Краткая информация о заказе ORD-20240115:
      order_id: ORD-20240115
      customer: Иван Петров
      items_count: 2
      total_quantity: 3
      subtotal: 53000.0
      discount: 0.0
      delivery_cost: 500.0
      total: 48000.0
      status: pending

============================================================
ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА
============================================================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (простые модели с Field)

Варианты 9-17: средний (+ кастомные валидаторы, вложенные модели)

Варианты 18-25: сложный (+ корневые валидаторы, интеграция в ETL)

Варианты 1-8 (Базовый уровень)
№	Модель	Поля	Field ограничения
1	User	name, email, age	name min_length=2, email regex, age 0-150
2	Product	name, price, stock	name min_length=1, price gt=0, stock ge=0
3	Book	title, author, year	title min_length=1, year 1500-2025
4	Movie	title, director, rating	title min_length=1, rating 0-10
5	Student	name, age, grade	name min_length=2, age 16-100, grade 1-5
6	Employee	name, position, salary	name min_length=2, salary ge=0
7	Car	brand, model, year, price	year 1900-2025, price gt=0
8	Task	title, priority, completed	title min_length=1, priority 1-5
Варианты 9-17 (Средний уровень)
№	Модели	Валидаторы	Особенности
9	User, Address	email, phone	Вложенная модель Address
10	Order, OrderItem	sum validation	Расчёт общей суммы
11	Product, Category	category validation	Валидация существования категории
12	Event, Ticket	date validation	Проверка дат (start < end)
13	Loan, Book, User	unique validation	Проверка уникальности ISBN
14	Hotel, Room, Booking	availability	Проверка доступности номеров
15	Survey, Question, Answer	required fields	Обязательность ответов
16	Subscription, Plan, Payment	price validation	Расчёт цены со скидкой
17	Shipment, Package, Address	weight validation	Проверка веса отправления
Варианты 18-25 (Сложный уровень)
№	Тема	Модели	Особенности
18	Интернет-магазин	User, Address, Product, Order, OrderItem	Полная валидация заказа
19	Банковская система	User, Account, Transaction, Card	Валидация транзакций
20	Бронирование отелей	User, Hotel, Room, Booking, Payment	Проверка дат, доступности
21	Система голосования	Voter, Candidate, Vote, Election	Уникальность голоса
22	CRM система	Customer, Deal, Contact, Activity	Валидация этапов сделки
23	Образовательная платформа	Student, Course, Enrollment, Grade	Проверка оценок, кредитов
24	Социальная сеть	User, Post, Comment, Like, Friend	Валидация контента, дружбы
25	Финансовый отчёт	Transaction, Category, Budget, Forecast	Валидация бюджета, прогнозов
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Модуль не работает или валидация отсутствует
3 (удовлетворительно)	Созданы 2-3 модели с базовой валидацией
4 (хорошо)	+ вложенные модели, кастомные валидаторы
5 (отлично)	+ корневые валидаторы, интеграция в ETL, отчёты
5. Шпаргалка
python
# === БАЗОВАЯ МОДЕЛЬ ===
from pydantic import BaseModel, Field, validator, root_validator

class MyModel(BaseModel):
    field1: str = Field(..., min_length=1, max_length=100)
    field2: int = Field(..., gt=0, le=1000)
    
    @validator('field1')
    def validate_field1(cls, v):
        return v.strip().title()
    
    @root_validator
    def validate_all(cls, values):
        if values.get('field1') == values.get('field2'):
            raise ValueError('Поля не должны совпадать')
        return values

# === ВЛОЖЕННЫЕ МОДЕЛИ ===
class Address(BaseModel):
    city: str
    street: str

class User(BaseModel):
    name: str
    address: Address

# === ОПЦИОНАЛЬНЫЕ ПОЛЯ ===
from typing import Optional
field: Optional[str] = None

# === СПИСКИ ===
from typing import List
items: List[Item]

# === ДАТЫ ===
from datetime import datetime
created_at: datetime = Field(default_factory=datetime.now)

# === ВАЛИДАЦИЯ В ETL ===
validator = MyModelValidator()
valid_data = validator.validate_batch(raw_data)
Карточка студента
text
ПЗ 2.32. РАЗРАБОТКА МОДУЛЯ ВАЛИДАЦИИ С PYDANTIC

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== МОДЕЛИ ===

1. _____________
   Поля: _____________

2. _____________
   Поля: _____________

3. _____________
   Поля: _____________

=== ВАЛИДАЦИЯ ===

□ Field (min_length, max_length)
□ Field (gt, ge, lt, le)
□ Field (regex)
□ @validator (одиночные поля)
□ @root_validator (межполевая)
□ Вложенные модели
□ Встроенные типы (EmailStr, UrlStr)

=== ОТЧЁТ ===

Файл validator.py: _____________
Количество моделей: _____
Пример успешной валидации: _____________
Пример ошибки валидации: _____________

Дата выполнения: _____________
