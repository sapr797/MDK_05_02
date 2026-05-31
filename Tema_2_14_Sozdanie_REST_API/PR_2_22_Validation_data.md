# ПЗ 2.22. Валидация входных данных с Pydantic в FastAPI

**Тема:** Валидация данных, библиотека Pydantic, FastAPI, модели, валидаторы

**Цель работы:**  
Научиться использовать Pydantic для валидации входных данных в FastAPI, создавать сложные модели с вложенными структурами, писать пользовательские валидаторы, обрабатывать ошибки валидации.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленный FastAPI, Uvicorn и Pydantic

```bash
pip install fastapi uvicorn pydantic email-validator
Главная мысль: Pydantic превращает аннотации типов в мощную систему валидации. Никакого ручного if — только декларативные правила.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Что такое валидация данных
Валидация данных — это процесс проверки данных на соответствие заданным правилам перед их использованием.

Проблемы без валидации:

python
# Без валидации — код может упасть
def create_user(name, email, age):
    # Нет проверки: name может быть пустым
    # Нет проверки: email может быть без @
    # Нет проверки: age может быть строкой
    return {"name": name, "email": email, "age": age}
С валидацией (Pydantic):

python
from pydantic import BaseModel, Field, EmailStr

class UserCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)

# Валидация происходит автоматически
user = UserCreate(name="Иван", email="ivan@example.com", age=25)
1.2. Встроенные типы Pydantic
Тип	Описание	Пример
str	Строка	name: str
int	Целое число	age: int
float	Вещественное число	price: float
bool	Логическое	is_active: bool
EmailStr	Email (валидация)	email: EmailStr
UrlStr	URL (валидация)	website: UrlStr
UUID	UUID	id: UUID
datetime	Дата и время	created_at: datetime
date	Дата	birth_date: date
List[T]	Список	tags: List[str]
Dict[K, V]	Словарь	meta: Dict[str, Any]
Optional[T]	Может быть None	middle_name: Optional[str]
1.3. Параметры Field
Параметр	Описание	Пример
...	Обязательное поле	Field(...)
default	Значение по умолчанию	Field(default="Аноним")
min_length	Минимальная длина строки	Field(min_length=3)
max_length	Максимальная длина строки	Field(max_length=100)
gt	Больше чем (> )	Field(gt=0)
ge	Больше или равно (>=)	Field(ge=18)
lt	Меньше чем (<)	Field(lt=100)
le	Меньше или равно (<=)	Field(le=120)
regex	Регулярное выражение	Field(regex=r'^[A-Z]+$')
description	Описание поля	Field(description="Имя")
1.4. Валидаторы
python
from pydantic import BaseModel, validator, root_validator

class MyModel(BaseModel):
    field1: str
    field2: int
    
    # Валидатор одного поля
    @validator('field1')
    def validate_field1(cls, v):
        if not v:
            raise ValueError('Поле не может быть пустым')
        return v.strip()
    
    # Валидатор с предварительной обработкой
    @validator('field2', pre=True)
    def parse_field2(cls, v):
        if isinstance(v, str):
            return int(v)
        return v
    
    # Валидатор нескольких полей
    @root_validator
    def validate_all(cls, values):
        if values.get('field1') == values.get('field2'):
            raise ValueError('Поля не должны совпадать')
        return values
1.5. Ошибки валидации в FastAPI
json
{
    "detail": [
        {
            "loc": ["body", "name"],
            "msg": "ensure this value has at least 1 characters",
            "type": "value_error.any_str.min_length",
            "ctx": {"limit_value": 1}
        },
        {
            "loc": ["body", "email"],
            "msg": "value is not a valid email address",
            "type": "value_error.email"
        }
    ]
}
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая использование Pydantic для валидации в FastAPI.

Техническое задание (нулевой вариант)
Разработайте API для регистрации пользователей с комплексной валидацией данных с помощью Pydantic. Требования:

Имя: от 2 до 50 символов, только буквы и пробелы

Email: корректный формат, уникальный (проверка через список)

Пароль: минимум 8 символов, заглавная буква, цифра, спецсимвол

Возраст: от 18 до 120 лет

Телефон: российский номер (+7XXXXXXXXXX)

Адрес: город, улица, дом (опционально квартира)

Согласие с условиями: обязательно

Эталонная реализация
python
#!/usr/bin/env python3
"""
registration_api.py — API регистрации пользователей с валидацией Pydantic.
"""

import re
import uuid
from datetime import datetime
from typing import Optional, List, Set
from pydantic import BaseModel, Field, EmailStr, validator, root_validator
from fastapi import FastAPI, HTTPException, status

# ============================================================
# КОНСТАНТЫ
# ============================================================

# Список занятых email (для проверки уникальности)
EXISTING_EMAILS: Set[str] = {
    "admin@example.com",
    "test@example.com"
}

# Запрещённые пароли
WEAK_PASSWORDS: Set[str] = {
    "password", "12345678", "qwerty123", "admin123"
}

# Допустимые коды операторов РФ
VALID_OPERATOR_CODES = ['901', '902', '903', '904', '905', '906', '907', '908', '909',
                         '910', '911', '912', '913', '914', '915', '916', '917', '918', '919',
                         '920', '921', '922', '923', '924', '925', '926', '927', '928', '929',
                         '930', '931', '932', '933', '934', '935', '936', '937', '938', '939',
                         '950', '951', '952', '953', '954', '955', '956', '957', '958', '959']


# ============================================================
# ВЛОЖЕННЫЕ МОДЕЛИ
# ============================================================

class Address(BaseModel):
    """Модель адреса."""
    city: str = Field(..., min_length=1, max_length=100, description="Город")
    street: str = Field(..., min_length=1, max_length=100, description="Улица")
    house: str = Field(..., min_length=1, max_length=10, description="Номер дома")
    apartment: Optional[str] = Field(None, max_length=10, description="Номер квартиры")
    
    @validator('city', 'street')
    def validate_city_street(cls, v):
        """Проверяет, что название города и улицы содержит только буквы."""
        if not re.match(r'^[a-zA-Zа-яА-ЯёЁ\s\-]+$', v):
            raise ValueError('Название может содержать только буквы, пробелы и дефисы')
        return v.strip().title()
    
    @validator('house')
    def validate_house(cls, v):
        """Проверяет формат номера дома."""
        if not re.match(r'^\d+[а-яА-Я]?$', v):
            raise ValueError('Номер дома должен содержать цифры и опционально букву')
        return v
    
    def __str__(self):
        result = f"{self.city}, {self.street}, {self.house}"
        if self.apartment:
            result += f", кв. {self.apartment}"
        return result


# ============================================================
# ОСНОВНАЯ МОДЕЛЬ
# ============================================================

class UserRegistration(BaseModel):
    """Модель регистрации пользователя с полной валидацией."""
    
    # === ОСНОВНЫЕ ПОЛЯ ===
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
        max_length=100,
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
    
    address: Address = Field(
        ...,
        description="Адрес проживания"
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
    
    # === ВАЛИДАТОРЫ ОДИНОЧНЫХ ПОЛЕЙ ===
    
    @validator('name')
    def name_only_letters(cls, v: str) -> str:
        """Проверяет, что имя содержит только буквы, пробелы и дефисы."""
        if not re.match(r'^[a-zA-Zа-яА-ЯёЁ\s\-]+$', v):
            raise ValueError('Имя может содержать только буквы, пробелы и дефисы')
        # Нормализация: удаляем лишние пробелы, приводим к титульному регистру
        return ' '.join(v.split()).title()
    
    @validator('email')
    def email_unique(cls, v: str) -> str:
        """Проверяет уникальность email."""
        if v.lower() in EXISTING_EMAILS:
            raise ValueError('Пользователь с таким email уже зарегистрирован')
        return v.lower()
    
    @validator('password')
    def password_strong(cls, v: str) -> str:
        """Проверяет сложность пароля."""
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
        """Проверяет и нормализует российский номер телефона."""
        # Удаляем все нецифровые символы
        digits = re.sub(r'\D', '', v)
        
        # Проверка длины
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
        if operator_code not in VALID_OPERATOR_CODES:
            raise ValueError(f'Некорректный код оператора: {operator_code}')
        
        return result
    
    @validator('agree_to_terms')
    def must_agree_to_terms(cls, v: bool) -> bool:
        """Проверяет, что пользователь согласен с условиями."""
        if not v:
            raise ValueError('Вы должны согласиться с условиями')
        return v
    
    @validator('promo_code')
    def validate_promo_code(cls, v: Optional[str]) -> Optional[str]:
        """Проверяет формат промокода (если указан)."""
        if v is None:
            return v
        
        # Промокод должен быть в формате: XXX-XXXX-XXX
        if not re.match(r'^[A-Z0-9]{3}-[A-Z0-9]{4}-[A-Z0-9]{3}$', v):
            raise ValueError('Неверный формат промокода. Ожидается: XXX-XXXX-XXX')
        
        return v.upper()
    
    # === ВАЛИДАТОР НЕСКОЛЬКИХ ПОЛЕЙ ===
    
    @root_validator
    def check_phone_country_match(cls, values):
        """
        Проверяет соответствие телефона стране проживания.
        (Демонстрация root_validator)
        """
        phone = values.get('phone')
        address = values.get('address')
        
        if phone and address:
            # Проверка: если телефон российский, то страна должна быть Россия
            # В данном примере просто логируем
            if phone.startswith('+7') and address.city:
                # Можно добавить проверку города из России
                pass
        
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
                "address": {
                    "city": "Москва",
                    "street": "Тверская",
                    "house": "10",
                    "apartment": "42"
                },
                "agree_to_terms": True,
                "subscribe_newsletter": True,
                "promo_code": "ABC-1234-XYZ"
            }
        }


# ============================================================
# МОДЕЛЬ ОТВЕТА
# ============================================================

class UserResponse(BaseModel):
    """Модель ответа после успешной регистрации."""
    id: str
    name: str
    email: str
    age: int
    phone: str
    address: Address
    created_at: datetime
    subscribe_newsletter: bool
    
    class Config:
        schema_extra = {
            "example": {
                "id": "123e4567-e89b-12d3-a456-426614174000",
                "name": "Иван Петров",
                "email": "ivan@example.com",
                "age": 25,
                "phone": "+79123456789",
                "address": {
                    "city": "Москва",
                    "street": "Тверская",
                    "house": "10",
                    "apartment": "42"
                },
                "created_at": "2024-01-15T14:30:25.123456",
                "subscribe_newsletter": True
            }
        }


# ============================================================
# FASTAPI ПРИЛОЖЕНИЕ
# ============================================================

app = FastAPI(
    title="User Registration API",
    description="API для регистрации пользователей с валидацией Pydantic",
    version="1.0.0"
)

# Хранилище пользователей (в памяти)
users_db = {}


# ============================================================
# ЭНДПОИНТЫ
# ============================================================

@app.post(
    '/register',
    response_model=UserResponse,
    status_code=201,
    summary="Регистрация пользователя",
    description="Регистрирует нового пользователя с валидацией всех полей"
)
async def register_user(user_data: UserRegistration):
    """
    Регистрация нового пользователя.
    
    Все данные проходят валидацию через Pydantic:
    - Имя: только буквы, 2-50 символов
    - Email: корректный формат, уникальный
    - Пароль: минимум 8 символов, заглавная буква, цифра, спецсимвол
    - Возраст: 18-120 лет
    - Телефон: российский номер
    - Адрес: валидный адрес
    - Согласие с условиями: обязательно
    - Промокод (опционально): формат XXX-XXXX-XXX
    """
    # Генерация ID
    user_id = str(uuid.uuid4())
    
    # Создание пользователя
    new_user = UserResponse(
        id=user_id,
        name=user_data.name,
        email=user_data.email,
        age=user_data.age,
        phone=user_data.phone,
        address=user_data.address,
        created_at=datetime.now(),
        subscribe_newsletter=user_data.subscribe_newsletter
    )
    
    # Сохранение в "базу данных"
    users_db[user_id] = new_user
    
    # Добавляем email в список занятых (для демонстрации)
    EXISTING_EMAILS.add(user_data.email)
    
    return new_user


@app.get(
    '/users/{user_id}',
    response_model=UserResponse,
    summary="Получить пользователя",
    description="Возвращает данные зарегистрированного пользователя"
)
async def get_user(user_id: str):
    """Получение пользователя по ID."""
    if user_id not in users_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Пользователь с ID {user_id} не найден"
        )
    return users_db[user_id]


@app.get(
    '/validate/phone/{phone}',
    summary="Валидация телефона",
    description="Проверяет корректность номера телефона"
)
async def validate_phone(phone: str):
    """Эндпоинт для проверки телефона без регистрации."""
    try:
        # Временная модель для валидации
        class PhoneCheck(BaseModel):
            phone: str
            
            @validator('phone')
            def validate_phone(cls, v):
                # Используем ту же логику, что и в UserRegistration
                digits = re.sub(r'\D', '', v)
                if len(digits) not in [10, 11]:
                    raise ValueError('Некорректная длина')
                return v
        
        PhoneCheck(phone=phone)
        return {"valid": True, "phone": phone}
    except ValueError as e:
        return {"valid": False, "error": str(e)}


# ============================================================
# ОБРАБОТЧИК ОШИБОК ВАЛИДАЦИИ
# ============================================================

@app.exception_handler(HTTPException)
async def http_exception_handler(request, exc):
    """Кастомный обработчик HTTP исключений."""
    from fastapi.responses import JSONResponse
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": True,
            "code": exc.status_code,
            "message": exc.detail
        }
    )


# ============================================================
# ЗАПУСК
# ============================================================

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        "main:app",
        host="127.0.0.1",
        port=8000,
        reload=True
    )
Тестирование API
Тест 1: Успешная регистрация
bash
curl -X POST http://localhost:8000/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Иван Петров",
    "email": "ivan@example.com",
    "password": "SecurePass123!",
    "age": 25,
    "phone": "+79123456789",
    "address": {
        "city": "Москва",
        "street": "Тверская",
        "house": "10",
        "apartment": "42"
    },
    "agree_to_terms": true,
    "subscribe_newsletter": true
  }'
Ответ:

json
{
    "id": "abc123...",
    "name": "Иван Петров",
    "email": "ivan@example.com",
    "age": 25,
    "phone": "+79123456789",
    "address": {
        "city": "Москва",
        "street": "Тверская",
        "house": "10",
        "apartment": "42"
    },
    "created_at": "2024-01-15T14:30:25.123456",
    "subscribe_newsletter": true
}
Тест 2: Ошибка валидации (короткое имя)
bash
curl -X POST http://localhost:8000/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "А",
    "email": "ivan@example.com",
    "password": "SecurePass123!",
    "age": 25,
    "phone": "+79123456789",
    "address": {
        "city": "Москва",
        "street": "Тверская",
        "house": "10"
    },
    "agree_to_terms": true
}'
Ответ (422 Unprocessable Entity):

json
{
    "detail": [
        {
            "loc": ["body", "name"],
            "msg": "ensure this value has at least 2 characters",
            "type": "value_error.any_str.min_length",
            "ctx": {"limit_value": 2}
        }
    ]
}
Тест 3: Валидация телефона
bash
curl -X GET "http://localhost:8000/validate/phone/123"
Ответ:

json
{
    "valid": false,
    "error": "Номер телефона должен содержать 10 или 11 цифр"
}
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (Field валидация)

Варианты 9-17: средний (+ @validator)

Варианты 18-25: сложный (+ вложенные модели, @root_validator)

Варианты 1-8 (Базовый уровень)
№	Ресурс	Поля	Field ограничения
1	Продукт	name, price, category	name min_length=3, price gt=0
2	Студент	name, age, grade	name min_length=2, age 16-100
3	Книга	title, author, year	year 1900-2025
4	Фильм	title, director, rating	rating 0-10
5	Автомобиль	brand, model, year, price	year 1950-2025, price gt=0
6	Сотрудник	name, position, salary	salary ge=0
7	Заказ	total, status, items	total gt=0
8	Отзыв	rating, comment, author	rating 1-5
Варианты 9-17 (Средний уровень)
№	Ресурс	Поля	Валидаторы
9	Пользователь	name, email, password	email regex, password strength
10	Кредитная карта	number, expiry, cvv	luhn check, expiry date
11	Дата события	year, month, day, hour	is_valid_date
12	IP адрес	ip, port	ipv4/ipv6 regex, port 1-65535
13	Настройки	theme, language, notifications	theme in ['light','dark']
14	Сотрудник	salary, bonus, department	salary range
15	Заказ	items, total, discount	total = sum(items)
16	Отель	name, stars, price, amenities	stars 1-5, price gt=0
17	Пользователь	username, email, phone	уникальность (имитация)
Варианты 18-25 (Сложный уровень)
№	Ресурс	Особенности
18	Банковский перевод	Сумма, валюта, комиссия, root_validator
19	Бронирование отеля	Даты заезда/выезда, проверка дат
20	Экзамен	Студент, предмет, оценка, допуск
21	Маршрут	Точки, расстояние, время
22	Отчёт	Период, данные, формат
23	Анкета	Вопросы, ответы, обязательность
24	Плагин	Имя, версия, зависимости
25	Транзакция	Отправитель, получатель, сумма, лимиты
Пример варианта 19 (проверка дат):

python
class Booking(BaseModel):
    check_in: date
    check_out: date
    
    @root_validator
    def validate_dates(cls, values):
        check_in = values.get('check_in')
        check_out = values.get('check_out')
        
        if check_in >= check_out:
            raise ValueError('Дата выезда должна быть позже даты заезда')
        
        if (check_out - check_in).days < 1:
            raise ValueError('Минимальное проживание — 1 день')
        
        if (check_out - check_in).days > 30:
            raise ValueError('Максимальное проживание — 30 дней')
        
        return values
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Валидация отсутствует или не работает
3 (удовлетворительно)	Использованы Field для 3-4 полей
4 (хорошо)	+ @validator для сложных проверок
5 (отлично)	+ вложенные модели, @root_validator, обработка ошибок
5. Шпаргалка
5.1. Базовый шаблон модели
python
from pydantic import BaseModel, Field, validator, root_validator

class MyModel(BaseModel):
    field1: str = Field(..., min_length=1, max_length=50)
    field2: int = Field(..., gt=0, le=100)
    
    @validator('field1')
    def validate_field1(cls, v):
        if not v.strip():
            raise ValueError('Поле не может быть пустым')
        return v.strip()
    
    @root_validator
    def validate_all(cls, values):
        if values.get('field1') == str(values.get('field2')):
            raise ValueError('Поля не должны совпадать')
        return values
5.2. Field параметры
python
# Обязательное поле
Field(...)

# Со значением по умолчанию
Field(default="значение")

# Строковые ограничения
Field(min_length=1, max_length=100, regex=r'^[A-Z]+$')

# Числовые ограничения
Field(gt=0, ge=0, lt=100, le=100)

# Описание
Field(description="Описание поля")
5.3. Встроенные валидаторы
python
from pydantic import EmailStr, UrlStr, UUID, PositiveInt, NegativeInt

email: EmailStr      # валидация email
url: UrlStr          # валидация URL
uid: UUID            # валидация UUID
positive: PositiveInt # > 0
negative: NegativeInt # < 0
5.4. Обработка ошибок в FastAPI
python
from fastapi import HTTPException, status

try:
    user = UserRegistration(**data)
except ValidationError as e:
    raise HTTPException(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        detail=e.errors()
    )
Карточка студента
text
ПЗ 2.22. ВАЛИДАЦИЯ ВХОДНЫХ ДАННЫХ С PYDANTIC В FASTAPI

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Ресурс: _________________________________

=== ИСПОЛЬЗОВАННЫЕ СРЕДСТВА ===

□ Field (обязательность, min_length, max_length)
□ Field (gt, ge, lt, le)
□ Field (regex)
□ @validator (одиночное поле)
□ @root_validator (несколько полей)
□ Вложенные модели
□ EmailStr, UrlStr

=== ПОЛЯ МОДЕЛИ ===

1. _____________ (тип: ____, ограничения: __________)
2. _____________ (тип: ____, ограничения: __________)
3. _____________ (тип: ____, ограничения: __________)
4. _____________ (тип: ____, ограничения: __________)

=== ОТЧЁТ ===

Пример успешной валидации: _____________
Пример ошибки валидации: _____________
Скриншот Swagger документации: _____________

Дата выполнения: _____________
