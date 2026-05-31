# ПЗ 2.18. Написание проверок валидности с pydantic

**Тема:** Валидация данных, библиотека pydantic, модели, валидаторы

**Цель работы:**  
Научиться создавать модели данных с помощью Pydantic, писать собственные валидаторы, обрабатывать ошибки валидации, использовать встроенные типы и Field для проверки данных.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленная библиотека Pydantic

```bash
pip install pydantic pydantic[email]
Главная мысль: Никогда не доверяйте внешним данным. Каждое поле — проверяй, каждое значение — валидируй.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Основные возможности Pydantic
Возможность	Описание
Автоматическая валидация типов	Проверяет соответствие типов данных
Преобразование типов	Автоматически приводит строки к числам
Field	Дополнительные ограничения: min, max, regex
Валидаторы	Пользовательские проверки с помощью @validator
Вложенные модели	Поддержка сложных иерархических структур
Ошибки валидации	Подробные сообщения о том, что не так
1.2. Схема валидации в Pydantic
text
┌─────────────────────────────────────────────────────────────────┐
│                    ВХОДНЫЕ ДАННЫЕ (dict, JSON)                   │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│              ШАГ 1: ВАЛИДАЦИЯ ТИПОВ                              │
│              (автоматическая, Field)                             │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│              ШАГ 2: ПРЕОБРАЗОВАНИЕ ТИПОВ                         │
│              (int ← str, float ← str, datetime ← str)            │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│              ШАГ 3: ВАЛИДАТОРЫ с pre=True                        │
│              (предварительная обработка)                         │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│              ШАГ 4: ОСНОВНЫЕ ВАЛИДАТОРЫ (@validator)             │
│              (пользовательские проверки)                         │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│              ШАГ 5: ROOT ВАЛИДАТОР (@root_validator)             │
│              (проверка взаимосвязей полей)                       │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    МОДЕЛЬ PYDANTIC                               │
└─────────────────────────────────────────────────────────────────┘
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая создание проверок валидности с Pydantic.

Техническое задание (нулевой вариант)
Разработайте модель UserRegistration для валидации данных регистрации пользователя. Требования:

Имя: от 2 до 50 символов, только буквы и пробелы

Email: корректный формат, уникальный (проверка через список существующих)

Пароль: минимум 8 символов, заглавная буква, цифра, спецсимвол

Возраст: от 18 до 120 лет

Телефон: российский номер (+7 или 8, 10 цифр)

Согласие с условиями: обязательно True

Подписка на рассылку: по умолчанию False

Эталонная реализация
python
#!/usr/bin/env python3
"""
user_registration.py — Модель для валидации данных регистрации пользователя.
"""

import re
from typing import Optional, List, Set
from pydantic import BaseModel, Field, validator, root_validator, EmailStr, ValidationError


# ============================================================
# КОНСТАНТЫ
# ============================================================

# Список занятых email (для проверки уникальности)
EXISTING_EMAILS: Set[str] = {
    "admin@example.com",
    "test@example.com",
    "ivan@example.com"
}

# Запрещённые пароли (слишком простые)
WEAK_PASSWORDS: Set[str] = {
    "password", "12345678", "qwerty123", "admin123", "letmein"
}


# ============================================================
# МОДЕЛЬ ДАННЫХ
# ============================================================

class UserRegistration(BaseModel):
    """Модель для валидации данных регистрации пользователя."""
    
    # === ПОЛЯ С Field ===
    name: str = Field(
        ...,
        min_length=2,
        max_length=50,
        description="Полное имя пользователя"
    )
    
    email: EmailStr = Field(
        ...,
        description="Электронная почта (уникальная)"
    )
    
    password: str = Field(
        ...,
        min_length=8,
        description="Пароль (минимум 8 символов)"
    )
    
    age: int = Field(
        ...,
        ge=18,
        le=120,
        description="Возраст (от 18 до 120 лет)"
    )
    
    phone: str = Field(
        ...,
        description="Номер телефона (российский формат)"
    )
    
    agree_to_terms: bool = Field(
        ...,
        description="Согласие с условиями (обязательно)"
    )
    
    subscribe_newsletter: bool = Field(
        default=False,
        description="Подписка на рассылку"
    )
    
    promo_code: Optional[str] = Field(
        None,
        max_length=20,
        description="Промокод (опционально)"
    )
    
    # === ВАЛИДАТОРЫ ===
    
    @validator('name')
    def name_only_letters_and_spaces(cls, v: str) -> str:
        """
        Проверяет, что имя содержит только буквы и пробелы.
        """
        if not re.match(r'^[a-zA-Zа-яА-ЯёЁ\s]+$', v):
            raise ValueError('Имя может содержать только буквы и пробелы')
        # Нормализация: удаляем лишние пробелы, приводим к титульному регистру
        return ' '.join(v.split()).title()
    
    @validator('email')
    def email_unique(cls, v: str) -> str:
        """
        Проверяет уникальность email.
        """
        if v.lower() in EXISTING_EMAILS:
            raise ValueError('Пользователь с таким email уже зарегистрирован')
        return v.lower()
    
    @validator('password')
    def password_strong(cls, v: str) -> str:
        """
        Проверяет сложность пароля.
        """
        # Проверка на запрещённые пароли
        if v.lower() in WEAK_PASSWORDS:
            raise ValueError('Пароль слишком простой. Выберите более сложный пароль')
        
        # Проверка на наличие заглавной буквы
        if not re.search(r'[A-ZА-Я]', v):
            raise ValueError('Пароль должен содержать хотя бы одну заглавную букву')
        
        # Проверка на наличие цифры
        if not re.search(r'\d', v):
            raise ValueError('Пароль должен содержать хотя бы одну цифру')
        
        # Проверка на наличие спецсимвола
        if not re.search(r'[!@#$%^&*(),.?":{}|<>]', v):
            raise ValueError('Пароль должен содержать хотя бы один специальный символ')
        
        return v
    
    @validator('phone')
    def validate_phone(cls, v: str) -> str:
        """
        Проверяет и нормализует российский номер телефона.
        """
        # Удаляем все нецифровые символы
        digits = re.sub(r'\D', '', v)
        
        # Проверка длины (10 или 11 цифр)
        if len(digits) not in [10, 11]:
            raise ValueError('Номер телефона должен содержать 10 или 11 цифр')
        
        # Нормализация: приводим к формату +7XXXXXXXXXX
        if len(digits) == 11:
            if digits[0] == '8':
                digits = '7' + digits[1:]
            elif digits[0] != '7':
                raise ValueError('Номер должен начинаться с +7, 8 или 7')
        
        result = f"+7{digits[-10:]}"
        
        # Проверка кода оператора
        operator_code = digits[-10:-7]
        valid_codes = ['901', '902', '903', '904', '905', '906', '907', '908', '909',
                       '910', '911', '912', '913', '914', '915', '916', '917', '918', '919',
                       '920', '921', '922', '923', '924', '925', '926', '927', '928', '929',
                       '930', '931', '932', '933', '934', '935', '936', '937', '938', '939',
                       '950', '951', '952', '953', '954', '955', '956', '957', '958', '959']
        
        if operator_code not in valid_codes:
            raise ValueError(f'Некорректный код оператора: {operator_code}')
        
        return result
    
    @validator('promo_code')
    def validate_promo_code(cls, v: Optional[str]) -> Optional[str]:
        """
        Проверяет формат промокода (если указан).
        """
        if v is None:
            return v
        
        # Промокод должен быть в формате: XXX-XXXX-XXX
        if not re.match(r'^[A-Z0-9]{3}-[A-Z0-9]{4}-[A-Z0-9]{3}$', v):
            raise ValueError('Неверный формат промокода. Ожидается: XXX-XXXX-XXX')
        
        return v.upper()
    
    @root_validator
    def check_phone_country_match(cls, values):
        """
        Проверяет соответствие телефона стране (дополнительная бизнес-логика).
        """
        # Если есть поле country, можно было бы проверять соответствие
        # В данном примере просто логируем
        return values
    
    # === КОНФИГУРАЦИЯ ===
    class Config:
        title = "User Registration"
        description = "Модель для валидации данных регистрации пользователя"
        schema_extra = {
            "example": {
                "name": "Иван Петров",
                "email": "ivan@example.com",
                "password": "SecurePass123!",
                "age": 25,
                "phone": "+79123456789",
                "agree_to_terms": True,
                "subscribe_newsletter": True,
                "promo_code": "ABC-1234-XYZ"
            }
        }


# ============================================================
# ФУНКЦИЯ ДЛЯ ОБРАБОТКИ ВАЛИДАЦИИ
# ============================================================

def validate_user_data(data: dict) -> dict:
    """
    Валидирует данные пользователя и возвращает результат.
    
    Args:
        data: Словарь с данными пользователя
    
    Returns:
        Словарь с результатами валидации
    """
    result = {
        "is_valid": False,
        "errors": [],
        "validated_data": None
    }
    
    try:
        user = UserRegistration(**data)
        result["is_valid"] = True
        result["validated_data"] = user.dict()
        result["message"] = "Данные успешно проверены"
        
    except ValidationError as e:
        result["errors"] = []
        for error in e.errors():
            field = '.'.join(error.get('loc', []))
            msg = error.get('msg', 'Unknown error')
            result["errors"].append({
                "field": field,
                "message": msg
            })
        result["message"] = f"Найдено {len(result['errors'])} ошибок валидации"
    
    return result


# ============================================================
# ДЕМОНСТРАЦИЯ
# ============================================================

def main():
    """Демонстрация работы модели валидации."""
    print("=" * 70)
    print("ВАЛИДАЦИЯ ДАННЫХ ПОЛЬЗОВАТЕЛЯ С PYDANTIC")
    print("=" * 70)
    
    # Тестовые данные
    test_cases = [
        {
            "name": "Иван Петров",
            "email": "ivan_new@example.com",
            "password": "SecurePass123!",
            "age": 25,
            "phone": "+79123456789",
            "agree_to_terms": True,
            "subscribe_newsletter": True,
            "promo_code": "ABC-1234-XYZ"
        },
        {
            "name": "J0hn Doe",  # Содержит цифры
            "email": "test@example.com",  # Уже существует
            "password": "weak",
            "age": 15,  # Меньше 18
            "phone": "123",
            "agree_to_terms": False,  # Не согласен
        },
        {
            "name": "Анна",
            "email": "anna@example.com",
            "password": "NoDigits!",
            "age": 30,
            "phone": "+71234567890",
            "agree_to_terms": True,
        }
    ]
    
    for i, test_data in enumerate(test_cases, 1):
        print(f"\n--- ТЕСТ #{i} ---")
        print(f"Входные данные: {test_data}")
        
        result = validate_user_data(test_data)
        
        if result["is_valid"]:
            print(f"✅ {result['message']}")
            print(f"   Валидированные данные: {result['validated_data']}")
        else:
            print(f"❌ {result['message']}")
            for error in result["errors"]:
                print(f"   • {error['field']}: {error['message']}")
    
    # Демонстрация преобразования в JSON-схему
    print("\n" + "=" * 70)
    print("JSON СХЕМА МОДЕЛИ")
    print("=" * 70)
    print(UserRegistration.schema_json(indent=2))


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
======================================================================
ВАЛИДАЦИЯ ДАННЫХ ПОЛЬЗОВАТЕЛЯ С PYDANTIC
======================================================================

--- ТЕСТ #1 ---
Входные данные: {'name': 'Иван Петров', 'email': 'ivan_new@example.com', 'password': 'SecurePass123!', 'age': 25, 'phone': '+79123456789', 'agree_to_terms': True, 'subscribe_newsletter': True, 'promo_code': 'ABC-1234-XYZ'}
✅ Данные успешно проверены
   Валидированные данные: {'name': 'Иван Петров', 'email': 'ivan_new@example.com', 'password': 'SecurePass123!', 'age': 25, 'phone': '+79123456789', 'agree_to_terms': True, 'subscribe_newsletter': True, 'promo_code': 'ABC-1234-XYZ'}

--- ТЕСТ #2 ---
Входные данные: {'name': 'J0hn Doe', 'email': 'test@example.com', 'password': 'weak', 'age': 15, 'phone': '123', 'agree_to_terms': False}
❌ Найдено 6 ошибок валидации
   • name: Имя может содержать только буквы и пробелы
   • email: Пользователь с таким email уже зарегистрирован
   • password: Пароль должен содержать хотя бы одну заглавную букву
   • age: ensure this value is greater than or equal to 18
   • phone: Номер телефона должен содержать 10 или 11 цифр
   • agree_to_terms: field required

--- ТЕСТ #3 ---
Входные данные: {'name': 'Анна', 'email': 'anna@example.com', 'password': 'NoDigits!', 'age': 30, 'phone': '+71234567890', 'agree_to_terms': True}
❌ Найдено 1 ошибок валидации
   • password: Пароль должен содержать хотя бы одну цифру

======================================================================
JSON СХЕМА МОДЕЛИ
======================================================================
{
  "title": "User Registration",
  "description": "Модель для валидации данных регистрации пользователя",
  "type": "object",
  "properties": {
    ...
  }
}
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (простые модели с Field)

Варианты 9-17: средний (с валидаторами, регулярными выражениями)

Варианты 18-25: сложный (root_validator, вложенные модели, пользовательские типы)

Варианты 1-8 (Базовый уровень)
№	Предметная область	Поля модели	Field ограничения
1	Товар (Product)	id, name, price, stock	name min=3, price>0, stock>=0
2	Студент (Student)	name, age, grade, group	age 17-60, grade 1-5
3	Книга (Book)	title, author, year, isbn	year 1500-2025, isbn regex
4	Фильм (Movie)	title, director, year, rating	year 1900-2025, rating 0-10
5	Заказ (Order)	id, total, status, created_at	status in ['pending', 'completed']
6	Пользователь (User)	username, email, age	username alphanumeric
7	Адрес (Address)	city, street, house, zip	zip digits, house positive
8	Автомобиль (Car)	brand, model, year, price	year>1900, price>0
Пример варианта 1:

python
class Product(BaseModel):
    id: int = Field(..., gt=0)
    name: str = Field(..., min_length=3, max_length=100)
    price: float = Field(..., gt=0)
    stock: int = Field(..., ge=0)
Варианты 9-17 (Средний уровень)
№	Предметная область	Поля модели	Валидаторы
9	Регистрация пользователя	name, email, password, confirm_password	password strength, password match
10	Кредитная карта	number, expiry, cvv	luhn check, expiry date, cvv length
11	Дата события	year, month, day, hour, minute	is_valid_date, is_valid_time
12	Пароль	password, confirm	strength check, match
13	IP адрес	ip, port	ipv4/ipv6 regex, port range
14	Настройки приложения	theme, language, notifications	theme in ['light','dark']
15	Заказ с доставкой	delivery_type, address, time	delivery_type match address
16	Сотрудник	name, position, salary, hire_date	salary range, date not future
17	Отель	name, stars, price, facilities	stars 1-5, price>0
Пример варианта 9:

python
class UserRegister(BaseModel):
    email: EmailStr
    password: str
    confirm_password: str
    
    @validator('password')
    def password_strong(cls, v):
        # проверка сложности
        return v
    
    @root_validator
    def passwords_match(cls, values):
        if values.get('password') != values.get('confirm_password'):
            raise ValueError('Пароли не совпадают')
        return values
Варианты 18-25 (Сложный уровень)
№	Предметная область	Особенности
18	Банковский перевод	Сумма, валюта, комиссия, root_validator
19	Расписание занятий	День недели, время, аудитория, пересечения
20	Экзаменационная ведомость	Студент, предмет, оценка, допуск
21	Маршрут доставки	Точки, расстояние, время, ограничения
22	Отчёт о продажах	Период, товары, скидки, итоги
23	Анкета опроса	Вопросы, типы ответов, обязательность
24	Плагин системы	Имя, версия, зависимости, совместимость
25	Бронирование билетов	Пассажир, рейс, место, багаж
Пример варианта 18:

python
class Transfer(BaseModel):
    amount: float
    currency: str
    fee_percent: float = 1.5
    
    @root_validator
    def calculate_total(cls, values):
        amount = values.get('amount', 0)
        fee_percent = values.get('fee_percent', 0)
        values['total'] = amount + amount * fee_percent / 100
        return values
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Модель не работает или содержит критические ошибки
3 (удовлетворительно)	Модель создана, но валидация неполная (2-3 поля)
4 (хорошо)	Модель создана, есть Field и простые валидаторы
5 (отлично)	Полная модель с Field, валидаторами, root_validator и обработкой ошибок
5. Шпаргалка
5.1. Быстрый шаблон
python
from pydantic import BaseModel, Field, validator, root_validator

class MyModel(BaseModel):
    field1: str = Field(..., min_length=1, max_length=100)
    field2: int = Field(..., gt=0, le=1000)
    
    @validator('field1')
    def validate_field1(cls, v):
        if not condition:
            raise ValueError('Ошибка')
        return v
    
    @root_validator
    def validate_all(cls, values):
        if values.get('field1') == values.get('field2'):
            raise ValueError('Поля не должны совпадать')
        return values
5.2. Основные Field параметры
python
Field(..., min_length=1)           # обязательное, мин. длина
Field('default', max_length=10)    # со значением по умолчанию
Field(..., gt=0, le=100)           # 0 < x <= 100
Field(..., ge=0, lt=100)           # 0 <= x < 100
Field(..., regex='^[A-Z]+$')       # только заглавные буквы
Field(default_factory=list)        # фабрика значений
5.3. Встроенные типы
python
from pydantic import EmailStr, UrlStr, UUID, PositiveInt, NegativeInt, PaymentCardNumber
5.4. Обработка ошибок
python
from pydantic import ValidationError

try:
    model = MyModel(**data)
except ValidationError as e:
    for error in e.errors():
        print(f"{error['loc']}: {error['msg']}")
Карточка студента
text
ПЗ 2.18. НАПИСАНИЕ ПРОВЕРОК ВАЛИДНОСТИ С PYDANTIC

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== МОДЕЛЬ ===

Название модели: _____________
Количество полей: _____
Обязательных полей: _____
Опциональных полей: _____

=== FIELD ===

□ min_length / max_length
□ gt / ge / lt / le
□ regex
□ default / default_factory

=== ВАЛИДАТОРЫ ===

□ @validator (одиночные поля)
□ @root_validator (несколько полей)
□ pre=True валидаторы
□ each_item=True

=== ОТЧЁТ ===

Файл модели: _____________
Пример успешной валидации: _____________
Пример ошибки валидации: _____________

Дата выполнения: _____________
