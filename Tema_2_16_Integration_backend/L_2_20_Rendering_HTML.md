# Тема 2.20. Рендеринг HTML на сервере (Jinja2)

**Цель лекции:**  
Изучить шаблонизатор Jinja2 для генерации HTML на сервере, освоить синтаксис шаблонов, наследование, фильтры, макросы и интеграцию с FastAPI/Flask.

> Главная мысль: **Jinja2 отделяет логику от представления, позволяя создавать динамические HTML-страницы без смешивания кода и разметки.**

---

## Содержание

1. [Введение в шаблонизаторы](#1-введение-в-шаблонизаторы)
2. [Установка и настройка Jinja2](#2-установка-и-настройка-jinja2)
3. [Синтаксис шаблонов](#3-синтаксис-шаблонов)
4. [Переменные и фильтры](#4-переменные-и-фильтры)
5. [Условные конструкции и циклы](#5-условные-конструкции-и-циклы)
6. [Наследование шаблонов](#6-наследование-шаблонов)
7. [Макросы и импорты](#7-макросы-и-импорты)
8. [Интеграция с FastAPI](#8-интеграция-с-fastapi)
9. [Практические примеры](#9-практические-примеры)
10. [Контрольные вопросы](#10-контрольные-вопросы)
11. [Практическое задание](#11-практическое-задание)
12. [Шпаргалка](#12-шпаргалка)

---

## 1. Введение в шаблонизаторы

### 1.1. Что такое шаблонизатор

**Шаблонизатор** — это инструмент для генерации HTML-страниц на основе шаблонов и динамических данных.
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ Шаблон │ │ Данные │ │ Результат │
│ (template) │ + │ (context) │ = │ (HTML) │
│ │ │ │ │ │
│ <h1>{{ title }}</h1> │ title="Привет" │ │ <h1>Привет</h1> │
└─────────────────┘ └─────────────────┘ └─────────────────┘

text

### 1.2. Зачем нужны шаблонизаторы

| Без шаблонизатора | С шаблонизатором |
|-------------------|------------------|
| HTML встроен в Python-код | HTML отдельно от кода |
| Трудно поддерживать | Лёгкая поддержка |
| Опасно (XSS уязвимости) | Автоматическое экранирование |
| Сложно переиспользовать | Наследование и макросы |

### 1.3. Jinja2 — основной шаблонизатор Python

**Jinja2** — современный, быстрый и безопасный шаблонизатор для Python.

```python
# Без шаблонизатора (плохо)
def user_page(user):
    return f"""
    <html>
        <head><title>{user['name']}</title></head>
        <body>
            <h1>Привет, {user['name']}!</h1>
            <p>Email: {user['email']}</p>
        </body>
    </html>
    """

# С Jinja2 (хорошо)
from jinja2 import Template

template = Template("""
<html>
    <head><title>{{ name }}</title></head>
    <body>
        <h1>Привет, {{ name }}!</h1>
        <p>Email: {{ email }}</p>
    </body>
</html>
""")

html = template.render(name="Иван", email="ivan@example.com")
2. Установка и настройка Jinja2
2.1. Установка
bash
pip install jinja2
2.2. Базовое использование
python
from jinja2 import Template

# Простой шаблон
template = Template("Hello {{ name }}!")
result = template.render(name="World")
print(result)  # Hello World!

# Шаблон с несколькими переменными
template = Template("""
Имя: {{ user.name }}
Email: {{ user.email }}
Возраст: {{ user.age }}
""")

user = {"name": "Иван", "email": "ivan@example.com", "age": 25}
result = template.render(user=user)
print(result)
2.3. Загрузка шаблонов из файлов
python
from jinja2 import Environment, FileSystemLoader

# Создаём окружение с указанием папки с шаблонами
env = Environment(loader=FileSystemLoader('templates'))

# Загружаем шаблон из файла
template = env.get_template('user.html')

# Рендерим с данными
html = template.render(name="Иван", email="ivan@example.com")
Структура проекта:

text
project/
├── templates/
│   ├── base.html
│   ├── user.html
│   └── users_list.html
└── app.py
3. Синтаксис шаблонов
3.1. Разделители Jinja2
Разделитель	Назначение	Пример
{{ ... }}	Вывод переменной	{{ user.name }}
{% ... %}	Управляющие конструкции	{% if %} {% for %}
{# ... #}	Комментарии	{# Это комментарий #}
3.2. Пример шаблона
html
<!-- templates/user.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{{ title }} | Мой сайт</title>
</head>
<body>
    {# Это комментарий — не попадёт в HTML #}
    
    <h1>Привет, {{ name }}!</h1>
    
    {% if is_admin %}
        <p>У вас есть права администратора</p>
    {% else %}
        <p>Вы обычный пользователь</p>
    {% endif %}
    
    <h2>Ваши задачи:</h2>
    <ul>
    {% for task in tasks %}
        <li>{{ task.title }} 
            {% if task.completed %}✅{% else %}⏳{% endif %}
        </li>
    {% endfor %}
    </ul>
</body>
</html>
4. Переменные и фильтры
4.1. Доступ к переменным
python
# Передача простых переменных
template.render(name="Иван", age=25)

# Доступ в шаблоне
{{ name }}
{{ age }}

# Передача словаря
user = {"name": "Иван", "email": "ivan@example.com"}
template.render(user=user)

# Доступ в шаблоне
{{ user.name }}
{{ user.email }}
{{ user['name'] }}

# Передача списка
users = [{"name": "Иван"}, {"name": "Мария"}]
template.render(users=users)

# Доступ в шаблоне
{% for user in users %}
    {{ user.name }}
{% endfor %}
4.2. Фильтры
Фильтры изменяют вывод переменной:

jinja2
{{ value|default("значение по умолчанию") }}
{{ name|capitalize }}
{{ name|upper }}
{{ name|lower }}
{{ name|title }}
{{ text|trim }}
{{ text|length }}
{{ number|round(2) }}
{{ date|date("Y-m-d") }}
{{ html|safe }}  {# Без экранирования HTML #}
{{ text|escape }} {# Экранирование HTML #}
Примеры:

jinja2
{{ "hello world"|title }}        {# Hello World #}
{{ "HELLO"|lower }}              {# hello #}
{{ "  text  "|trim }}            {# text #}
{{ 3.14159|round(2) }}           {# 3.14 #}
{{ 1000000|format_number }}      {# 1,000,000 #}
4.3. Цепочки фильтров
jinja2
{{ name|trim|title|escape }}
{{ text|truncate(50)|upper }}
4.4. Встроенные фильтры
Фильтр	Описание	Пример
default	Значение по умолчанию	{{ value|default("—") }}
length	Длина	{{ items|length }}
join	Объединение списка	{{ tags|join(", ") }}
first	Первый элемент	{{ items|first }}
last	Последний элемент	{{ items|last }}
sort	Сортировка	{{ items|sort }}
reverse	Обратный порядок	{{ items|reverse }}
unique	Уникальные значения	{{ items|unique }}
batch	Разбивка на группы	{{ items|batch(3) }}
5. Условные конструкции и циклы
5.1. Условный оператор if
jinja2
{% if user.is_active %}
    <span class="active">Активен</span>
{% elif user.is_banned %}
    <span class="banned">Заблокирован</span>
{% else %}
    <span class="inactive">Неактивен</span>
{% endif %}
jinja2
{% if user and user.age >= 18 %}
    <p>Доступ разрешён</p>
{% endif %}
5.2. Цикл for
jinja2
<!-- Простой цикл по списку -->
<ul>
{% for item in items %}
    <li>{{ item }}</li>
{% endfor %}
</ul>
jinja2
<!-- Цикл по словарю -->
<dl>
{% for key, value in user.items() %}
    <dt>{{ key }}</dt>
    <dd>{{ value }}</dd>
{% endfor %}
</dl>
5.3. Специальные переменные в цикле
Переменная	Описание
loop.index	Текущая итерация (с 1)
loop.index0	Текущая итерация (с 0)
loop.first	Первая итерация?
loop.last	Последняя итерация?
loop.length	Общее количество
loop.cycle	Циклическое переключение
jinja2
<ul>
{% for item in items %}
    <li class="{{ loop.cycle('odd', 'even') }}">
        {{ loop.index }}. {{ item }}
        {% if loop.first %} (первый){% endif %}
        {% if loop.last %} (последний){% endif %}
    </li>
{% endfor %}
</ul>
5.4. Цикл с условием
jinja2
{# Пропуск пустых элементов #}
<ul>
{% for item in items if item %}
    <li>{{ item }}</li>
{% endfor %}
</ul>
jinja2
{# else для пустого списка #}
<ul>
{% for item in items %}
    <li>{{ item }}</li>
{% else %}
    <li>Нет элементов</li>
{% endfor %}
</ul>
6. Наследование шаблонов
6.1. Базовый шаблон (base.html)
html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Мой сайт{% endblock %}</title>
    <link rel="stylesheet" href="/static/css/style.css">
    
    {% block head %}{% endblock %}
</head>
<body>
    <header>
        <nav>
            <a href="/">Главная</a>
            <a href="/about">О нас</a>
            <a href="/contact">Контакты</a>
        </nav>
    </header>
    
    <main>
        {% block content %}
        {% endblock %}
    </main>
    
    <footer>
        <p>&copy; 2024 Мой сайт</p>
    </footer>
    
    <script src="/static/js/script.js"></script>
    {% block scripts %}{% endblock %}
</body>
</html>
6.2. Дочерний шаблон (home.html)
html
{% extends "base.html" %}

{% block title %}Главная страница{% endblock %}

{% block content %}
<h1>Добро пожаловать!</h1>

<p>Это главная страница нашего сайта.</p>

<div class="posts">
    {% for post in posts %}
    <article class="post">
        <h2>{{ post.title }}</h2>
        <p class="date">{{ post.created_at|date("d.m.Y") }}</p>
        <p>{{ post.excerpt }}</p>
        <a href="/posts/{{ post.id }}">Читать далее →</a>
    </article>
    {% endfor %}
</div>
{% endblock %}
6.3. Вложенные блоки
jinja2
{% extends "base.html" %}

{# Переопределение блока с сохранением содержимого родителя #}
{% block content %}
    {{ super() }}
    <p>Дополнительный контент после родительского</p>
{% endblock %}
7. Макросы и импорты
7.1. Макросы (макеты)
Макросы — это переиспользуемые фрагменты шаблонов.

jinja2
<!-- macros/forms.html -->
{% macro input(name, value='', type='text', label='', class='') %}
<div class="form-group">
    {% if label %}
        <label for="{{ name }}">{{ label }}</label>
    {% endif %}
    <input type="{{ type }}" 
           name="{{ name }}" 
           id="{{ name }}" 
           value="{{ value }}"
           class="{{ class }}">
</div>
{% endmacro %}

{% macro button(text, type='button', class='btn') %}
<button type="{{ type }}" class="{{ class }}">{{ text }}</button>
{% endmacro %}
7.2. Импорт макросов
jinja2
{% import "macros/forms.html" as forms %}

<form method="post">
    {{ forms.input('name', label='Имя') }}
    {{ forms.input('email', label='Email', type='email') }}
    {{ forms.input('age', label='Возраст', type='number') }}
    
    {{ forms.button('Отправить', type='submit', class='btn-primary') }}
</form>
7.3. include для вставки файлов
jinja2
{# Вставка содержимого другого шаблона #}
{% include "header.html" %}

<main>
    {% block content %}{% endblock %}
</main>

{% include "footer.html" %}
8. Интеграция с FastAPI
8.1. Настройка Jinja2 в FastAPI
python
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles

app = FastAPI()

# Подключаем статические файлы (CSS, JS)
app.mount("/static", StaticFiles(directory="static"), name="static")

# Настраиваем шаблоны
templates = Jinja2Templates(directory="templates")

# Добавляем пользовательские фильтры
def format_date(value):
    return value.strftime("%d.%m.%Y %H:%M")

templates.env.filters["format_date"] = format_date
8.2. Использование в эндпоинтах
python
@app.get("/")
async def home(request: Request):
    # Данные для шаблона
    context = {
        "request": request,  # Обязательно для FastAPI!
        "title": "Главная",
        "posts": get_posts(),
        "user": get_current_user()
    }
    return templates.TemplateResponse("home.html", context)

@app.get("/users/{user_id}")
async def user_profile(request: Request, user_id: int):
    user = get_user(user_id)
    
    context = {
        "request": request,
        "user": user,
        "posts": get_user_posts(user_id),
        "is_following": check_following(user_id)
    }
    
    return templates.TemplateResponse("profile.html", context)
8.3. Обработка форм с FastAPI + Jinja2
python
from fastapi import Form, HTTPException

@app.get("/register")
async def register_form(request: Request):
    return templates.TemplateResponse("register.html", {"request": request})

@app.post("/register")
async def register(
    request: Request,
    name: str = Form(...),
    email: str = Form(...),
    password: str = Form(...)
):
    errors = {}
    
    # Валидация
    if len(name) < 2:
        errors["name"] = "Имя слишком короткое"
    
    if "@" not in email:
        errors["email"] = "Неверный email"
    
    if errors:
        return templates.TemplateResponse(
            "register.html", 
            {"request": request, "errors": errors, "name": name, "email": email}
        )
    
    # Сохранение пользователя...
    
    # Редирект на страницу успеха
    from fastapi.responses import RedirectResponse
    return RedirectResponse(url="/login", status_code=303)
9. Практические примеры
9.1. Блог с Jinja2 (полный пример)
python
# main.py
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles
from datetime import datetime

app = FastAPI(title="Мой блог")
app.mount("/static", StaticFiles(directory="static"), name="static")
templates = Jinja2Templates(directory="templates")

# Добавляем фильтр для форматирования даты
def format_date(date):
    return date.strftime("%d %B %Y")

templates.env.filters["format_date"] = format_date

# Данные (имитация базы данных)
posts = [
    {
        "id": 1,
        "title": "Первая запись",
        "content": "Содержание первой записи...",
        "created_at": datetime(2024, 1, 15),
        "author": "Иван",
        "tags": ["python", "web"]
    },
    {
        "id": 2,
        "title": "Вторая запись",
        "content": "Содержание второй записи...",
        "created_at": datetime(2024, 1, 20),
        "author": "Мария",
        "tags": ["fastapi", "jinja2"]
    }
]

@app.get("/")
async def home(request: Request):
    return templates.TemplateResponse(
        "index.html",
        {"request": request, "posts": posts}
    )

@app.get("/posts/{post_id}")
async def post_detail(request: Request, post_id: int):
    post = next((p for p in posts if p["id"] == post_id), None)
    if not post:
        return templates.TemplateResponse(
            "404.html", 
            {"request": request}, 
            status_code=404
        )
    
    return templates.TemplateResponse(
        "post.html",
        {"request": request, "post": post}
    )

@app.get("/about")
async def about(request: Request):
    return templates.TemplateResponse(
        "about.html",
        {"request": request}
    )

@app.get("/search")
async def search(request: Request, q: str = ""):
    results = []
    if q:
        results = [p for p in posts if q.lower() in p["title"].lower()]
    
    return templates.TemplateResponse(
        "search.html",
        {"request": request, "query": q, "results": results}
    )
9.2. Шаблон index.html
html
{% extends "base.html" %}

{% block title %}Главная страница{% endblock %}

{% block content %}
<div class="hero">
    <h1>Добро пожаловать в мой блог</h1>
    <p>Здесь я делюсь мыслями о программировании</p>
</div>

<div class="posts-list">
    <h2>Последние записи</h2>
    
    {% for post in posts %}
    <article class="post">
        <h3>
            <a href="/posts/{{ post.id }}">{{ post.title }}</a>
        </h3>
        <div class="post-meta">
            <span class="date">{{ post.created_at|format_date }}</span>
            <span class="author">Автор: {{ post.author }}</span>
        </div>
        <div class="post-excerpt">
            {{ post.content[:200] }}...
        </div>
        <div class="post-tags">
            {% for tag in post.tags %}
                <span class="tag">{{ tag }}</span>
            {% endfor %}
        </div>
        <a href="/posts/{{ post.id }}" class="read-more">Читать далее →</a>
    </article>
    {% else %}
        <p>Пока нет записей</p>
    {% endfor %}
</div>
{% endblock %}
9.3. Шаблон base.html
html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Мой блог{% endblock %}</title>
    <link href="/static/css/style.css" rel="stylesheet">
    {% block head %}{% endblock %}
</head>
<body>
    <header>
        <nav class="navbar">
            <div class="container">
                <a href="/" class="logo">Мой блог</a>
                <ul class="nav-links">
                    <li><a href="/">Главная</a></li>
                    <li><a href="/about">О сайте</a></li>
                    <li><a href="/search">Поиск</a></li>
                </ul>
            </div>
        </nav>
    </header>

    <main class="container">
        {% block content %}{% endblock %}
    </main>

    <footer>
        <div class="container">
            <p>&copy; 2024 Мой блог. Все права защищены.</p>
        </div>
    </footer>

    <script src="/static/js/script.js"></script>
    {% block scripts %}{% endblock %}
</body>
</html>
10. Контрольные вопросы
Что такое шаблонизатор и зачем он нужен?

Какой синтаксис используется в Jinja2 для вывода переменных?

Как написать условие if-else в шаблоне?

Как перебрать список элементов в цикле for?

Что такое фильтры? Приведите 3 примера.

Как создать базовый шаблон и наследовать его?

Как передать данные из FastAPI в шаблон?

Что такое макросы в Jinja2 и зачем они нужны?

Как подключить статические файлы (CSS, JS)?

Как обработать форму с FastAPI и Jinja2?

11. Практическое задание
Задание 1 (базовое)
Создайте шаблон user_profile.html для отображения профиля пользователя с полями: имя, email, возраст, город.

Задание 2 (среднее)
Создайте блог с:

Главной страницей (список постов)

Страницей поста

Боковой панелью с категориями

Формой для добавления комментариев

Задание 3 (сложное)
Разработайте полноценный сайт на FastAPI + Jinja2:

Регистрация и авторизация пользователей

CRUD для постов

Комментарии

Поиск

Пагинация

Админ-панель

12. Шпаргалка
jinja2
{# === ПЕРЕМЕННЫЕ === #}
{{ variable }}
{{ object.attribute }}
{{ dict.key }}
{{ list[index] }}

{# === ФИЛЬТРЫ === #}
{{ value|default("—") }}
{{ text|truncate(50) }}
{{ date|date("d.m.Y") }}
{{ html|safe }}
{{ text|escape }}

{# === УСЛОВИЯ === #}
{% if condition %}
{% elif other %}
{% else %}
{% endif %}

{# === ЦИКЛЫ === #}
{% for item in items %}
    {{ loop.index }}. {{ item }}
{% endfor %}

{# === НАСЛЕДОВАНИЕ === #}
{% extends "base.html" %}
{% block content %}{% endblock %}

{# === ВСТАВКА === #}
{% include "partial.html" %}

{# === МАКРОСЫ === #}
{% macro input(name) %}
    <input name="{{ name }}">
{% endmacro }
{{ input('username') }}
Итог лекции
Вы сегодня:

✅ Изучили шаблонизатор Jinja2 и его синтаксис

✅ Научились использовать переменные и фильтры

✅ Освоили условные конструкции и циклы

✅ Изучили наследование шаблонов и макросы

✅ Интегрировали Jinja2 с FastAPI

Теперь вы можете создавать полноценные динамические веб-сайты на Python!

