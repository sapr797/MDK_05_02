# Тема 2.17. Веб-фреймворки Python: Flask и FastAPI. Сравнение

**Цель лекции:**  
Изучить основные веб-фреймворки Python — Flask и FastAPI, понять их архитектуру, преимущества и недостатки, научиться выбирать подходящий фреймворк для разных типов проектов.

> Главная мысль: **Flask — для простоты и гибкости, FastAPI — для скорости и современного подхода. Выбор зависит от задачи.**

---

## Содержание

1. [Введение в веб-фреймворки](#1-введение-в-веб-фреймворки)
2. [Flask: основы и архитектура](#2-flask-основы-и-архитектура)
3. [FastAPI: основы и архитектура](#3-fastapi-основы-и-архитектура)
4. [Сравнение Flask и FastAPI](#4-сравнение-flask-и-fastapi)
5. [Практические примеры](#5-практические-примеры)
6. [Выбор фреймворка для проекта](#6-выбор-фреймворка-для-проекта)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Введение в веб-фреймворки

### 1.1. Что такое веб-фреймворк

**Веб-фреймворк** — это набор инструментов и библиотек для разработки веб-приложений.

**Основные компоненты веб-фреймворка:**

| Компонент | Назначение |
|-----------|------------|
| Маршрутизация | Связь URL с обработчиками |
| Шаблонизация | Генерация HTML страниц |
| Работа с БД | ORM, миграции |
| Middleware | Обработка запросов до и после |
| Аутентификация | Управление пользователями |
| Формы | Валидация и обработка данных |
| API | Создание REST API |

### 1.2. Популярные Python-фреймворки

| Фреймворк | Тип | Год | Основная характеристика |
|-----------|-----|-----|------------------------|
| **Django** | Полнофункциональный | 2005 | "Всё включено" |
| **Flask** | Микрофреймворк | 2010 | Гибкий, минималистичный |
| **FastAPI** | Современный | 2018 | Асинхронный, быстрый |
| **Falcon** | Минималистичный | 2013 | Для API, без UI |
| **aiohttp** | Асинхронный | 2013 | Полный контроль |

---

## 2. Flask: основы и архитектура

### 2.1. Что такое Flask

**Flask** — микрофреймворк для Python, разработанный Армином Ронахером. Он минималистичен и гибок, позволяя разработчику выбирать компоненты по своему усмотрению.

```bash
# Установка Flask
pip install flask
2.2. Минимальное приложение
python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World!'

@app.route('/user/<name>')
def greet_user(name):
    return f'Привет, {name}!'

if __name__ == '__main__':
    app.run(debug=True, port=5000)
Структура проекта Flask:

text
my_flask_project/
├── app.py                 # Главный файл приложения
├── requirements.txt       # Зависимости
├── static/               # Статические файлы (CSS, JS, изображения)
│   ├── css/
│   ├── js/
│   └── images/
├── templates/            # HTML-шаблоны (Jinja2)
│   ├── base.html
│   └── index.html
└── venv/                 # Виртуальное окружение
2.3. Маршрутизация в Flask
python
from flask import Flask, request, jsonify

app = Flask(__name__)

# GET запрос
@app.route('/api/users', methods=['GET'])
def get_users():
    return jsonify({"users": [1, 2, 3]})

# POST запрос
@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    return jsonify({"id": 1, "name": data['name']}), 201

# GET с параметром в URL
@app.route('/api/users/<int:user_id>')
def get_user(user_id):
    return jsonify({"id": user_id, "name": f"User {user_id}"})

# GET с query параметрами
@app.route('/api/search')
def search():
    query = request.args.get('q', '')
    page = request.args.get('page', 1, type=int)
    return jsonify({"query": query, "page": page})

# Динамические маршруты
@app.route('/posts/<slug>')
@app.route('/posts/<slug>/')
@app.route('/posts/<slug>/edit')
def post_handler(slug):
    return f"Post: {slug}"
2.4. Работа с шаблонами (Jinja2)
python
from flask import render_template

@app.route('/profile/<name>')
def profile(name):
    user = {
        'name': name,
        'email': f'{name}@example.com',
        'age': 25,
        'is_active': True,
        'skills': ['Python', 'Flask', 'SQL']
    }
    return render_template('profile.html', user=user)
Шаблон templates/profile.html:

html
<!DOCTYPE html>
<html>
<head>
    <title>Профиль</title>
</head>
<body>
    <h1>Профиль пользователя</h1>
    
    {% if user.is_active %}
        <p>Статус: Активен</p>
    {% else %}
        <p>Статус: Неактивен</p>
    {% endif %}
    
    <p>Имя: {{ user.name }}</p>
    <p>Email: {{ user.email }}</p>
    <p>Возраст: {{ user.age }}</p>
    
    <h3>Навыки:</h3>
    <ul>
    {% for skill in user.skills %}
        <li>{{ skill }}</li>
    {% endfor %}
    </ul>
</body>
</html>
2.5. Middleware в Flask
python
from flask import request, g
from functools import wraps
import time

# Декоратор для логирования времени выполнения
def time_logger(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__} выполнен за {elapsed:.3f} сек")
        return result
    return wrapper

# Декоратор для проверки авторизации
def login_required(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        token = request.headers.get('Authorization')
        if not token or token != 'secret-token':
            return jsonify({"error": "Unauthorized"}), 401
        return func(*args, **kwargs)
    return wrapper

# Использование
@app.route('/protected')
@login_required
@time_logger
def protected_route():
    return jsonify({"message": "Welcome!"})

# Before/After request
@app.before_request
def before_request():
    print(f"Request: {request.method} {request.path}")
    g.start_time = time.time()

@app.after_request
def after_request(response):
    elapsed = time.time() - g.start_time
    print(f"Response: {response.status_code} ({elapsed:.3f}s)")
    return response
2.6. Обработка ошибок в Flask
python
from flask import render_template

# Обработчик 404
@app.errorhandler(404)
def not_found(error):
    return jsonify({"error": "Not found"}), 404

# Обработчик 500
@app.errorhandler(500)
def internal_error(error):
    return jsonify({"error": "Internal server error"}), 500

# Обработчик с шаблоном
@app.errorhandler(404)
def not_found_template(error):
    return render_template('404.html'), 404
3. FastAPI: основы и архитектура
3.1. Что такое FastAPI
FastAPI — современный веб-фреймворк для создания API. Он основан на асинхронном программировании и использует аннотации типов Python.

bash
# Установка FastAPI и ASGI сервера
pip install fastapi uvicorn
3.2. Минимальное приложение
python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get('/')
def hello():
    return {"message": "Hello, World!"}

@app.get('/user/{name}')
def greet_user(name: str):
    return {"message": f"Привет, {name}!"}

# Запуск: uvicorn main:app --reload
Структура проекта FastAPI:

text
my_fastapi_project/
├── main.py                # Главный файл приложения
├── requirements.txt       # Зависимости
├── app/
│   ├── __init__.py
│   ├── main.py            # Фабрика приложения
│   ├── routers/           # Роутеры (вместо blueprint)
│   │   ├── __init__.py
│   │   ├── users.py
│   │   └── items.py
│   ├── models/            # Pydantic модели
│   │   ├── __init__.py
│   │   └── schemas.py
│   └── database.py        # Работа с БД
└── tests/
    └── test_main.py
3.3. Pydantic модели и валидация
python
from fastapi import FastAPI
from pydantic import BaseModel, Field
from typing import Optional, List

app = FastAPI()

# Модель для валидации
class UserCreate(BaseModel):
    name: str = Field(..., min_length=2, max_length=50)
    email: str
    age: int = Field(..., ge=18, le=120)
    is_active: bool = True
    tags: List[str] = []

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    age: int
    is_active: bool

# Использование моделей
@app.post('/users', response_model=UserResponse, status_code=201)
def create_user(user: UserCreate):
    # Автоматическая валидация user
    return {
        "id": 1,
        "name": user.name,
        "email": user.email,
        "age": user.age,
        "is_active": user.is_active
    }
3.4. Параметры запроса
python
from fastapi import FastAPI, Query, Path, Body
from typing import Optional

app = FastAPI()

# Query параметры
@app.get('/items')
def get_items(
    skip: int = Query(0, ge=0),
    limit: int = Query(10, le=100),
    q: Optional[str] = Query(None, min_length=1, max_length=50)
):
    return {"skip": skip, "limit": limit, "q": q}

# Path параметры
@app.get('/users/{user_id}')
def get_user(
    user_id: int = Path(..., gt=0, title="ID пользователя")
):
    return {"user_id": user_id}

# Body параметры
@app.post('/orders')
def create_order(
    order: dict = Body(..., example={"product": "Ноутбук", "quantity": 1})
):
    return order
3.5. Асинхронные эндпоинты
python
import asyncio
from fastapi import FastAPI

app = FastAPI()

# Синхронный эндпоинт
@app.get('/sync')
def sync_endpoint():
    return {"result": "Sync"}

# Асинхронный эндпоинт
@app.get('/async')
async def async_endpoint():
    await asyncio.sleep(1)
    return {"result": "Async"}

# Асинхронный с конкурентными задачами
@app.get('/parallel')
async def parallel_tasks():
    task1 = asyncio.create_task(fetch_data_1())
    task2 = asyncio.create_task(fetch_data_2())
    result1 = await task1
    result2 = await task2
    return {"data1": result1, "data2": result2}

async def fetch_data_1():
    await asyncio.sleep(0.5)
    return "Data 1"

async def fetch_data_2():
    await asyncio.sleep(0.5)
    return "Data 2"
3.6. Зависимости (Dependency Injection)
python
from fastapi import FastAPI, Depends, Header, HTTPException

app = FastAPI()

# Простая зависимость
def common_parameters(q: str = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@app.get('/items')
def get_items(params: dict = Depends(common_parameters)):
    return params

# Зависимость для аутентификации
async def get_current_user(authorization: str = Header(...)):
    if not authorization.startswith('Bearer '):
        raise HTTPException(status_code=401, detail="Invalid token")
    token = authorization.replace('Bearer ', '')
    # Здесь проверка токена
    return {"user_id": 1, "username": "ivan"}

@app.get('/profile')
def get_profile(user: dict = Depends(get_current_user)):
    return {"user": user}
3.7. Автоматическая документация
python
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    description="Это пример API",
    version="1.0.0",
    docs_url="/api/docs",
    redoc_url="/api/redoc"
)

@app.get('/')
def root():
    """Главный эндпоинт."""
    return {"message": "Hello"}

@app.get('/users/{user_id}', summary="Получить пользователя")
def get_user(user_id: int):
    """
    Получает информацию о пользователе по ID.
    
    - **user_id**: ID пользователя (целое число > 0)
    """
    return {"user_id": user_id}
4. Сравнение Flask и FastAPI
4.1. Таблица сравнения
Характеристика	Flask	FastAPI
Год выпуска	2010	2018
Тип	Микрофреймворк	Современный фреймворк
Производительность	Низкая	Очень высокая
Асинхронность	Нет (с WSGI)	Да (ASGI)
Валидация данных	Не встроена (Flask-WTF)	Pydantic (встроена)
Документация API	Нет (требуется расширение)	Автоматическая (Swagger)
Аннотации типов	Опционально	Обязательно
Dependency Injection	Нет	Встроена
Размер сообщества	Огромный	Растущий
Поддержка WebSocket	Нет	Да
ORM интеграция	SQLAlchemy, Peewee	SQLAlchemy (async)
4.2. Производительность (сравнение)
text
Тест: 10 000 запросов, простой эндпоинт

FastAPI:   ~15 000 req/sec
Flask:     ~ 3 000 req/sec
Django:    ~ 2 000 req/sec

FastAPI примерно в 5 раз быстрее Flask
4.3. Плюсы и минусы Flask
Плюсы	Минусы
✅ Простота изучения	❌ Низкая производительность
✅ Гибкость	❌ Нет встроенной валидации
✅ Огромное сообщество	❌ Сложная асинхронность
✅ Множество расширений	❌ Нет автоматической документации
✅ Стабильность	❌ Нет dependency injection
4.4. Плюсы и минусы FastAPI
Плюсы	Минусы
✅ Высокая производительность	❌ Молодой, меньше материалов
✅ Встроенная валидация	❌ Более крутая кривая обучения
✅ Автоматическая документация	❌ Меньше расширений
✅ Асинхронность "из коробки"	❌ Требует знания аннотаций типов
✅ Dependency Injection	❌ Меньше готовых решений
5. Практические примеры
5.1. CRUD API на Flask
python
# flask_crud.py
from flask import Flask, request, jsonify

app = Flask(__name__)

# Данные в памяти
users = [
    {"id": 1, "name": "Иван", "email": "ivan@example.com"},
    {"id": 2, "name": "Мария", "email": "maria@example.com"}
]

@app.route('/users', methods=['GET'])
def get_users():
    return jsonify(users)

@app.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = next((u for u in users if u['id'] == user_id), None)
    if user is None:
        return jsonify({"error": "User not found"}), 404
    return jsonify(user)

@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    new_id = max(u['id'] for u in users) + 1 if users else 1
    new_user = {"id": new_id, "name": data['name'], "email": data['email']}
    users.append(new_user)
    return jsonify(new_user), 201

@app.route('/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    user = next((u for u in users if u['id'] == user_id), None)
    if user is None:
        return jsonify({"error": "User not found"}), 404
    data = request.get_json()
    user.update({"name": data['name'], "email": data['email']})
    return jsonify(user)

@app.route('/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    global users
    user = next((u for u in users if u['id'] == user_id), None)
    if user is None:
        return jsonify({"error": "User not found"}), 404
    users = [u for u in users if u['id'] != user_id]
    return '', 204

if __name__ == '__main__':
    app.run(debug=True)
5.2. CRUD API на FastAPI
python
# fastapi_crud.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List

app = FastAPI()

# Модели
class UserCreate(BaseModel):
    name: str
    email: str

class UserResponse(UserCreate):
    id: int

# Данные в памяти
users: List[UserResponse] = [
    UserResponse(id=1, name="Иван", email="ivan@example.com"),
    UserResponse(id=2, name="Мария", email="maria@example.com")
]

@app.get('/users', response_model=List[UserResponse])
def get_users():
    return users

@app.get('/users/{user_id}', response_model=UserResponse)
def get_user(user_id: int):
    user = next((u for u in users if u.id == user_id), None)
    if user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.post('/users', response_model=UserResponse, status_code=201)
def create_user(user: UserCreate):
    new_id = max(u.id for u in users) + 1 if users else 1
    new_user = UserResponse(id=new_id, **user.dict())
    users.append(new_user)
    return new_user

@app.put('/users/{user_id}', response_model=UserResponse)
def update_user(user_id: int, user: UserCreate):
    existing = next((u for u in users if u.id == user_id), None)
    if existing is None:
        raise HTTPException(status_code=404, detail="User not found")
    existing.name = user.name
    existing.email = user.email
    return existing

@app.delete('/users/{user_id}', status_code=204)
def delete_user(user_id: int):
    global users
    user = next((u for u in users if u.id == user_id), None)
    if user is None:
        raise HTTPException(status_code=404, detail="User not found")
    users = [u for u in users if u.id != user_id]
6. Выбор фреймворка для проекта
6.1. Когда выбирать Flask
Критерий	Причина
Простой проект	Не нужна излишняя сложность
Монолитное приложение	С серверными страницами (HTML)
Ограниченное время	Быстрое прототипирование
Большое сообщество	Готовые решения и плагины
Стандартный веб-сайт	Блог, портфолио, корпоративный сайт
6.2. Когда выбирать FastAPI
Критерий	Причина
API-first проект	Только бэкенд, отдельный фронтенд
Высокая нагрузка	Нужна производительность
Асинхронные операции	Работа с внешними API, WebSocket
Микросервисы	Лёгкие, быстрые сервисы
Современные требования	Аннотации типов, автодокументация
6.3. Диаграмма выбора
text
                    ┌─────────────────┐
                    │  Какой проект?  │
                    └────────┬────────┘
                             │
            ┌────────────────┼────────────────┐
            │                │                │
            ▼                ▼                ▼
    ┌───────────┐    ┌───────────┐    ┌───────────┐
    │  Веб-сайт │    │   API     │    │ Микросервис│
    │ с HTML    │    │           │    │           │
    └─────┬─────┘    └─────┬─────┘    └─────┬─────┘
          │                │                │
          ▼                ▼                ▼
    ┌───────────┐    ┌───────────┐    ┌───────────┐
    │   Flask   │    │ FastAPI   │    │ FastAPI   │
    │   + Jinja │    │           │    │  + async  │
    └───────────┘    └───────────┘    └───────────┘
7. Контрольные вопросы
Чем Flask отличается от FastAPI?

Какой фреймворк производительнее и почему?

Что такое ASGI и чем он отличается от WSGI?

Как в FastAPI реализована валидация данных?

Как в Flask реализовать асинхронный эндпоинт?

Что такое dependency injection и в каком фреймворке он встроен?

Какой фреймворк лучше для создания высоконагруженного API?

Как в FastAPI создать автоматическую документацию?

Как в Flask работать с шаблонами?

Какой фреймворк выбрать для блога? Для API бэкенда?

8. Практическое задание
Задание 1 (базовое)
Создайте простое приложение на Flask с двумя маршрутами: / (главная) и /about (о сайте).

Задание 2 (среднее)
Создайте CRUD API для управления книгами на Flask и на FastAPI. Сравните объём кода и сложность реализации.

Задание 3 (сложное)
Создайте микросервис для конвертации валют на FastAPI с использованием асинхронного запроса к внешнему API.

9. Шпаргалка
python
# === FLASK ===
from flask import Flask
app = Flask(__name__)

@app.route('/path')
def handler():
    return {'key': 'value'}

app.run(debug=True)

# === FASTAPI ===
from fastapi import FastAPI
app = FastAPI()

@app.get('/path')
def handler():
    return {'key': 'value'}

# Запуск: uvicorn main:app --reload

# === ОСНОВНЫЕ ОТЛИЧИЯ ===
# Flask:      синхронный, WSGI, без валидации
# FastAPI:    асинхронный, ASGI, Pydantic валидация
Итог лекции
Вы сегодня:

✅ Изучили два популярных веб-фреймворка Python

✅ Поняли архитектуру и особенности Flask и FastAPI

✅ Узнали о преимуществах и недостатках каждого

✅ Научились выбирать фреймворк под конкретную задачу

✅ Создали CRUD API на обоих фреймворках

Теперь вы можете создавать веб-приложения и API на Python!

