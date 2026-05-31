# ПЗ 2.24. Разработка интерфейса для отображения данных из API через Jinja2

**Тема:** Веб-фреймворки, Jinja2 шаблонизация, интеграция с API, клиент-серверное взаимодействие

**Цель работы:**  
Научиться разрабатывать полноценные веб-интерфейсы для отображения данных из внешних API с использованием FastAPI и шаблонизатора Jinja2.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `fastapi`, `uvicorn`, `jinja2`, `requests`

```bash
pip install fastapi uvicorn jinja2 requests
Главная мысль: Современный сайт — это мост между API и пользователем. Jinja2 превращает JSON в красивые HTML-страницы.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Архитектура приложения
text
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Браузер   │ ──► │   FastAPI   │ ──► │   API       │
│   (HTML)    │ ◄── │   (сервер)  │ ◄── │   (данные)  │
└─────────────┘     └─────────────┘     └─────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   Jinja2    │
                    │  (шаблоны)  │
                    └─────────────┘
1.2. Компоненты системы
Компонент	Назначение
FastAPI	Веб-сервер, обработка запросов
Jinja2	Генерация HTML на основе данных
Requests	HTTP-клиент для внешних API
Статические файлы	CSS, JS, изображения
1.3. Структура проекта
text
my_web_app/
├── app.py                 # FastAPI приложение
├── requirements.txt       # Зависимости
├── static/               # Статические файлы
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── script.js
├── templates/            # Jinja2 шаблоны
│   ├── base.html
│   ├── index.html
│   ├── detail.html
│   └── search.html
└── services/             # Сервисы для работы с API
    └── api_client.py
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая разработку интерфейса для отображения данных из API.

Техническое задание (нулевой вариант)
Разработайте веб-приложение для отображения информации о пользователях и их постах из JSONPlaceholder API. Приложение должно иметь:

Главную страницу со списком пользователей

Страницу пользователя с его данными и списком постов

Страницу поста с содержимым и комментариями

Поиск постов по заголовку

Адаптивный интерфейс с CSS

Эталонная реализация
Файл services/api_client.py
python
"""
services/api_client.py — Клиент для работы с JSONPlaceholder API.
"""

import requests
from typing import List, Dict, Optional
import logging

logger = logging.getLogger(__name__)


class JSONPlaceholderClient:
    """Клиент для работы с JSONPlaceholder API."""
    
    BASE_URL = 'https://jsonplaceholder.typicode.com'
    
    def __init__(self, timeout: int = 10):
        self.timeout = timeout
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'MyWebApp/1.0',
            'Accept': 'application/json'
        })
    
    def _request(self, endpoint: str) -> Optional[Dict | List]:
        """Выполняет GET запрос к API."""
        url = f"{self.BASE_URL}/{endpoint.lstrip('/')}"
        
        try:
            response = self.session.get(url, timeout=self.timeout)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            logger.error(f"Ошибка запроса к {url}: {e}")
            return None
    
    def get_users(self) -> List[Dict]:
        """Получение списка всех пользователей."""
        result = self._request('/users')
        return result if isinstance(result, list) else []
    
    def get_user(self, user_id: int) -> Optional[Dict]:
        """Получение пользователя по ID."""
        return self._request(f'/users/{user_id}')
    
    def get_posts(self, user_id: Optional[int] = None) -> List[Dict]:
        """Получение постов (опционально по пользователю)."""
        if user_id:
            result = self._request(f'/posts?userId={user_id}')
        else:
            result = self._request('/posts')
        return result if isinstance(result, list) else []
    
    def get_post(self, post_id: int) -> Optional[Dict]:
        """Получение поста по ID."""
        return self._request(f'/posts/{post_id}')
    
    def get_comments(self, post_id: int) -> List[Dict]:
        """Получение комментариев к посту."""
        result = self._request(f'/posts/{post_id}/comments')
        return result if isinstance(result, list) else []
    
    def search_posts(self, query: str) -> List[Dict]:
        """Поиск постов по заголовку (локальная фильтрация)."""
        posts = self.get_posts()
        if not posts:
            return []
        
        query_lower = query.lower()
        return [
            post for post in posts
            if query_lower in post.get('title', '').lower()
            or query_lower in post.get('body', '').lower()
        ]


# Создаём глобальный экземпляр клиента
api_client = JSONPlaceholderClient()
Файл templates/base.html
html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Блог платформа{% endblock %}</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="/static/css/style.css">
    {% block head %}{% endblock %}
</head>
<body>
    <header class="header">
        <div class="container">
            <nav class="nav">
                <a href="/" class="logo">📝 Блог платформа</a>
                <div class="nav-links">
                    <a href="/">Главная</a>
                    <a href="/users">Пользователи</a>
                    <a href="/search">Поиск</a>
                </div>
            </nav>
        </div>
    </header>

    <main class="main">
        <div class="container">
            {% block content %}{% endblock %}
        </div>
    </main>

    <footer class="footer">
        <div class="container">
            <p>&copy; 2024 Блог платформа. Данные из JSONPlaceholder API.</p>
        </div>
    </footer>

    <script src="/static/js/script.js"></script>
    {% block scripts %}{% endblock %}
</body>
</html>
Файл templates/index.html
html
{% extends "base.html" %}

{% block title %}Главная | Блог платформа{% endblock %}

{% block content %}
<div class="hero">
    <h1>Добро пожаловать в Блог платформу</h1>
    <p>Здесь вы найдёте интересные посты и информацию о пользователях</p>
    <div class="hero-buttons">
        <a href="/users" class="btn btn-primary">Пользователи</a>
        <a href="/posts" class="btn btn-secondary">Все посты</a>
    </div>
</div>

<div class="section">
    <h2>Последние посты</h2>
    <div class="posts-grid">
        {% for post in recent_posts %}
        <div class="post-card">
            <h3 class="post-title">
                <a href="/posts/{{ post.id }}">{{ post.title }}</a>
            </h3>
            <p class="post-body">{{ post.body[:120] }}...</p>
            <div class="post-meta">
                <span>Автор: <a href="/users/{{ post.userId }}">Пользователь #{{ post.userId }}</a></span>
            </div>
        </div>
        {% endfor %}
    </div>
    <div class="text-center">
        <a href="/posts" class="btn btn-outline">Все посты →</a>
    </div>
</div>

<div class="section">
    <h2>Популярные пользователи</h2>
    <div class="users-grid">
        {% for user in users %}
        <div class="user-card">
            <h3 class="user-name">
                <a href="/users/{{ user.id }}">{{ user.name }}</a>
            </h3>
            <p class="user-email">{{ user.email }}</p>
            <p class="user-company">{{ user.company.name }}</p>
        </div>
        {% endfor %}
    </div>
</div>
{% endblock %}
Файл templates/users.html
html
{% extends "base.html" %}

{% block title %}Пользователи | Блог платформа{% endblock %}

{% block content %}
<h1>Пользователи</h1>

<div class="users-list">
    {% for user in users %}
    <div class="user-card-large">
        <div class="user-info">
            <h2>
                <a href="/users/{{ user.id }}">{{ user.name }}</a>
            </h2>
            <p><strong>Email:</strong> {{ user.email }}</p>
            <p><strong>Телефон:</strong> {{ user.phone }}</p>
            <p><strong>Сайт:</strong> <a href="http://{{ user.website }}" target="_blank">{{ user.website }}</a></p>
        </div>
        <div class="user-address">
            <strong>Адрес:</strong>
            <p>{{ user.address.street }}, {{ user.address.suite }}</p>
            <p>{{ user.address.city }}, {{ user.address.zipcode }}</p>
        </div>
        <div class="user-company">
            <strong>Компания:</strong>
            <p>{{ user.company.name }}</p>
            <p class="company-catchphrase">{{ user.company.catchPhrase }}</p>
        </div>
        <div class="user-posts-link">
            <a href="/users/{{ user.id }}/posts" class="btn btn-sm">Посты пользователя →</a>
        </div>
    </div>
    {% endfor %}
</div>
{% endblock %}
Файл templates/user_posts.html
html
{% extends "base.html" %}

{% block title %}{{ user.name }} — Посты | Блог платформа{% endblock %}

{% block content %}
<div class="user-header">
    <h1>{{ user.name }}</h1>
    <p class="user-email">{{ user.email }}</p>
    <div class="user-info-block">
        <div class="info-item">
            <span class="label">Телефон:</span>
            <span>{{ user.phone }}</span>
        </div>
        <div class="info-item">
            <span class="label">Сайт:</span>
            <span><a href="http://{{ user.website }}" target="_blank">{{ user.website }}</a></span>
        </div>
        <div class="info-item">
            <span class="label">Компания:</span>
            <span>{{ user.company.name }}</span>
        </div>
        <div class="info-item">
            <span class="label">Адрес:</span>
            <span>{{ user.address.city }}, {{ user.address.street }}</span>
        </div>
    </div>
</div>

<div class="section">
    <h2>Посты пользователя ({{ posts|length }})</h2>
    
    {% if posts %}
    <div class="posts-list">
        {% for post in posts %}
        <article class="post-item">
            <h3>
                <a href="/posts/{{ post.id }}">{{ post.title }}</a>
            </h3>
            <p class="post-excerpt">{{ post.body[:150] }}...</p>
            <div class="post-actions">
                <a href="/posts/{{ post.id }}" class="read-more">Читать →</a>
            </div>
        </article>
        {% endfor %}
    </div>
    {% else %}
    <p>У пользователя пока нет постов</p>
    {% endif %}
</div>

<div class="back-link">
    <a href="/users">← Назад к списку пользователей</a>
</div>
{% endblock %}
Файл templates/post_detail.html
html
{% extends "base.html" %}

{% block title %}{{ post.title }} | Блог платформа{% endblock %}

{% block content %}
<article class="post-full">
    <div class="post-header">
        <h1>{{ post.title }}</h1>
        <div class="post-meta">
            <span class="post-author">
                Автор: <a href="/users/{{ post.userId }}">Пользователь #{{ post.userId }}</a>
            </span>
        </div>
    </div>
    
    <div class="post-content">
        <p>{{ post.body }}</p>
    </div>
</article>

<div class="comments-section">
    <h2>Комментарии ({{ comments|length }})</h2>
    
    {% if comments %}
    <div class="comments-list">
        {% for comment in comments %}
        <div class="comment-card">
            <div class="comment-header">
                <strong class="comment-name">{{ comment.name }}</strong>
                <span class="comment-email">{{ comment.email }}</span>
            </div>
            <div class="comment-body">
                <p>{{ comment.body }}</p>
            </div>
        </div>
        {% endfor %}
    </div>
    {% else %}
    <p>Нет комментариев</p>
    {% endif %}
</div>

<div class="back-link">
    <a href="/users/{{ post.userId }}/posts">← Все посты пользователя</a>
</div>
{% endblock %}
Файл templates/search.html
html
{% extends "base.html" %}

{% block title %}Поиск | Блог платформа{% endblock %}

{% block content %}
<h1>Поиск постов</h1>

<form action="/search" method="get" class="search-form">
    <input type="text" 
           name="q" 
           value="{{ query }}" 
           placeholder="Введите слово для поиска..."
           class="search-input">
    <button type="submit" class="search-btn">🔍 Найти</button>
</form>

{% if query %}
    {% if results %}
        <div class="search-results">
            <h2>Результаты поиска: {{ results|length }}</h2>
            <div class="posts-list">
                {% for post in results %}
                <article class="post-item">
                    <h3>
                        <a href="/posts/{{ post.id }}">{{ post.title }}</a>
                    </h3>
                    <p class="post-excerpt">{{ post.body[:150] }}...</p>
                    <div class="post-meta">
                        <span>Автор: <a href="/users/{{ post.userId }}">Пользователь #{{ post.userId }}</a></span>
                    </div>
                </article>
                {% endfor %}
            </div>
        </div>
    {% else %}
        <div class="no-results">
            <p>По запросу "{{ query }}" ничего не найдено</p>
        </div>
    {% endif %}
{% endif %}
{% endblock %}
Файл app.py
python
"""
app.py — Основное FastAPI приложение.
"""

from fastapi import FastAPI, Request, HTTPException
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles
from services.api_client import api_client
import logging

# Настройка логирования
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Создание приложения
app = FastAPI(title="Blog Platform", description="Веб-интерфейс для JSONPlaceholder API")

# Подключение статических файлов
app.mount("/static", StaticFiles(directory="static"), name="static")

# Подключение шаблонов
templates = Jinja2Templates(directory="templates")

# Добавление фильтра для форматирования даты
def format_date(value):
    if not value:
        return ""
    return value[:10] if len(value) > 10 else value

templates.env.filters["format_date"] = format_date


# ============================================================
# ГЛАВНАЯ СТРАНИЦА
# ============================================================

@app.get("/")
async def home(request: Request):
    """Главная страница со списком пользователей и последними постами."""
    users = api_client.get_users()
    posts = api_client.get_posts()
    
    # Безопасное получение данных
    users = users[:6] if users else []
    recent_posts = posts[:6] if posts else []
    
    return templates.TemplateResponse(
        "index.html",
        {
            "request": request,
            "users": users,
            "recent_posts": recent_posts
        }
    )


# ============================================================
# ПОЛЬЗОВАТЕЛИ
# ============================================================

@app.get("/users")
async def users_list(request: Request):
    """Страница со списком всех пользователей."""
    users = api_client.get_users()
    
    if not users:
        raise HTTPException(status_code=500, detail="Не удалось загрузить пользователей")
    
    return templates.TemplateResponse(
        "users.html",
        {"request": request, "users": users}
    )


@app.get("/users/{user_id}")
async def user_detail(request: Request, user_id: int):
    """Страница пользователя."""
    user = api_client.get_user(user_id)
    
    if not user:
        raise HTTPException(status_code=404, detail="Пользователь не найден")
    
    return templates.TemplateResponse(
        "user_detail.html",
        {"request": request, "user": user}
    )


@app.get("/users/{user_id}/posts")
async def user_posts(request: Request, user_id: int):
    """Страница с постами пользователя."""
    user = api_client.get_user(user_id)
    posts = api_client.get_posts(user_id=user_id)
    
    if not user:
        raise HTTPException(status_code=404, detail="Пользователь не найден")
    
    posts = posts if posts else []
    
    return templates.TemplateResponse(
        "user_posts.html",
        {"request": request, "user": user, "posts": posts}
    )


# ============================================================
# ПОСТЫ
# ============================================================

@app.get("/posts")
async def posts_list(request: Request):
    """Страница со списком всех постов."""
    posts = api_client.get_posts()
    
    if not posts:
        raise HTTPException(status_code=500, detail="Не удалось загрузить посты")
    
    return templates.TemplateResponse(
        "posts.html",
        {"request": request, "posts": posts}
    )


@app.get("/posts/{post_id}")
async def post_detail(request: Request, post_id: int):
    """Страница поста с комментариями."""
    post = api_client.get_post(post_id)
    comments = api_client.get_comments(post_id)
    
    if not post:
        raise HTTPException(status_code=404, detail="Пост не найден")
    
    comments = comments if comments else []
    
    return templates.TemplateResponse(
        "post_detail.html",
        {"request": request, "post": post, "comments": comments}
    )


# ============================================================
# ПОИСК
# ============================================================

@app.get("/search")
async def search(request: Request, q: str = ""):
    """Страница поиска постов."""
    results = []
    if q:
        results = api_client.search_posts(q)
    
    return templates.TemplateResponse(
        "search.html",
        {"request": request, "query": q, "results": results}
    )


# ============================================================
# ОБРАБОТЧИКИ ОШИБОК
# ============================================================

@app.exception_handler(404)
async def not_found_handler(request: Request, exc):
    """Кастомная страница 404."""
    return templates.TemplateResponse(
        "404.html",
        {"request": request},
        status_code=404
    )


@app.exception_handler(500)
async def server_error_handler(request: Request, exc):
    """Кастомная страница 500."""
    return templates.TemplateResponse(
        "500.html",
        {"request": request},
        status_code=500
    )


# ============================================================
# ЗАПУСК
# ============================================================

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        "app:app",
        host="127.0.0.1",
        port=8000,
        reload=True
    )
Файл static/css/style.css
css
/* reset.css */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    line-height: 1.6;
    color: #333;
    background-color: #f5f5f5;
}

/* container */
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}

/* header */
.header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 20px 0;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 20px;
}

.logo {
    font-size: 24px;
    font-weight: 700;
    color: white;
    text-decoration: none;
}

.nav-links {
    display: flex;
    gap: 30px;
}

.nav-links a {
    color: white;
    text-decoration: none;
    opacity: 0.9;
    transition: opacity 0.3s;
}

.nav-links a:hover {
    opacity: 1;
}

/* main */
.main {
    min-height: calc(100vh - 200px);
    padding: 40px 0;
}

/* footer */
.footer {
    background: #2d3748;
    color: #a0aec0;
    text-align: center;
    padding: 20px 0;
    margin-top: 40px;
}

/* buttons */
.btn {
    display: inline-block;
    padding: 10px 20px;
    border-radius: 5px;
    text-decoration: none;
    transition: all 0.3s;
    cursor: pointer;
    border: none;
    font-size: 16px;
}

.btn-primary {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
}

.btn-primary:hover {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.btn-secondary {
    background: white;
    color: #667eea;
    border: 1px solid #667eea;
}

.btn-secondary:hover {
    background: #667eea;
    color: white;
}

.btn-outline {
    border: 1px solid #ddd;
    color: #666;
    background: white;
}

.btn-outline:hover {
    border-color: #667eea;
    color: #667eea;
}

.btn-sm {
    padding: 5px 15px;
    font-size: 14px;
}

/* hero */
.hero {
    text-align: center;
    padding: 60px 20px;
    background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
    border-radius: 20px;
    margin-bottom: 40px;
}

.hero h1 {
    font-size: 48px;
    margin-bottom: 20px;
    color: #333;
}

.hero p {
    font-size: 18px;
    color: #666;
    margin-bottom: 30px;
}

.hero-buttons {
    display: flex;
    gap: 20px;
    justify-content: center;
    flex-wrap: wrap;
}

/* sections */
.section {
    margin-bottom: 60px;
}

.section h2 {
    font-size: 28px;
    margin-bottom: 30px;
    padding-bottom: 10px;
    border-bottom: 3px solid #667eea;
    display: inline-block;
}

/* posts grid */
.posts-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
    gap: 30px;
    margin-bottom: 30px;
}

.post-card {
    background: white;
    border-radius: 10px;
    padding: 20px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    transition: transform 0.3s;
}

.post-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 5px 20px rgba(0,0,0,0.15);
}

.post-title {
    font-size: 18px;
    margin-bottom: 10px;
}

.post-title a {
    color: #333;
    text-decoration: none;
}

.post-title a:hover {
    color: #667eea;
}

.post-body {
    color: #666;
    font-size: 14px;
    line-height: 1.5;
    margin-bottom: 15px;
}

.post-meta {
    font-size: 12px;
    color: #999;
}

/* users grid */
.users-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 20px;
}

.user-card {
    background: white;
    border-radius: 10px;
    padding: 20px;
    text-align: center;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.user-name {
    margin-bottom: 10px;
}

.user-name a {
    color: #333;
    text-decoration: none;
}

.user-name a:hover {
    color: #667eea;
}

.user-email {
    color: #666;
    font-size: 14px;
    margin-bottom: 5px;
}

.user-company {
    color: #999;
    font-size: 12px;
}

/* users list */
.users-list {
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.user-card-large {
    background: white;
    border-radius: 10px;
    padding: 25px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    display: grid;
    grid-template-columns: 1fr 1fr 1fr auto;
    gap: 20px;
    align-items: start;
}

@media (max-width: 768px) {
    .user-card-large {
        grid-template-columns: 1fr;
        gap: 15px;
    }
}

/* user header */
.user-header {
    background: white;
    border-radius: 10px;
    padding: 30px;
    margin-bottom: 30px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.user-info-block {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 15px;
    margin-top: 20px;
}

.info-item {
    padding: 10px;
    background: #f8f9fa;
    border-radius: 5px;
}

.info-item .label {
    font-weight: 600;
    margin-right: 10px;
}

/* posts list */
.posts-list {
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.post-item {
    background: white;
    border-radius: 10px;
    padding: 20px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.post-item h3 {
    margin-bottom: 10px;
}

.post-item h3 a {
    color: #333;
    text-decoration: none;
}

.post-item h3 a:hover {
    color: #667eea;
}

.post-excerpt {
    color: #666;
    margin-bottom: 15px;
}

.read-more {
    color: #667eea;
    text-decoration: none;
    font-size: 14px;
}

.read-more:hover {
    text-decoration: underline;
}

/* post full */
.post-full {
    background: white;
    border-radius: 10px;
    padding: 30px;
    margin-bottom: 30px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.post-header {
    margin-bottom: 20px;
    padding-bottom: 20px;
    border-bottom: 1px solid #eee;
}

.post-header h1 {
    font-size: 32px;
    margin-bottom: 15px;
}

.post-content {
    line-height: 1.8;
    font-size: 16px;
}

/* comments */
.comments-section {
    background: white;
    border-radius: 10px;
    padding: 30px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.comments-list {
    margin-top: 20px;
}

.comment-card {
    padding: 15px;
    border-bottom: 1px solid #eee;
}

.comment-card:last-child {
    border-bottom: none;
}

.comment-header {
    margin-bottom: 10px;
}

.comment-name {
    font-weight: 600;
    margin-right: 10px;
}

.comment-email {
    color: #999;
    font-size: 12px;
}

.comment-body {
    color: #666;
}

/* search */
.search-form {
    display: flex;
    gap: 10px;
    margin-bottom: 30px;
}

.search-input {
    flex: 1;
    padding: 12px;
    border: 1px solid #ddd;
    border-radius: 5px;
    font-size: 16px;
}

.search-btn {
    padding: 12px 24px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.no-results {
    text-align: center;
    padding: 50px;
    background: white;
    border-radius: 10px;
}

/* back link */
.back-link {
    margin-top: 30px;
    text-align: center;
}

.back-link a {
    color: #667eea;
    text-decoration: none;
}

.back-link a:hover {
    text-decoration: underline;
}

.text-center {
    text-align: center;
}

/* error pages */
.error-page {
    text-align: center;
    padding: 80px 20px;
}

.error-code {
    font-size: 120px;
    font-weight: 700;
    color: #667eea;
    margin-bottom: 20px;
}

.error-message {
    font-size: 24px;
    color: #666;
    margin-bottom: 30px;
}
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (один ресурс, простой интерфейс)

Варианты 9-17: средний (несколько ресурсов, фильтрация)

Варианты 18-25: сложный (полноценное приложение с админкой)

Варианты 1-8 (Базовый уровень)
№	API	Ресурс	Функционал
1	JSONPlaceholder	Посты	Список, детальная страница
2	JSONPlaceholder	Пользователи	Список, карточки пользователей
3	JSONPlaceholder	Комментарии	Список по посту
4	PokéAPI	Покемоны	Список, детальная карточка
5	Rick & Morty	Персонажи	Галерея, фильтрация
6	Cat Facts	Факты	Случайный факт, список
7	Random User	Пользователи	Генерация профиля
8	Agify API	Возраст	Предсказание по имени
Варианты 9-17 (Средний уровень)
№	API	Ресурсы	Дополнительно
9	JSONPlaceholder	Пользователи + Посты	Страница пользователя с постами
10	JSONPlaceholder	Посты + Комментарии	Страница поста с комментариями
11	JSONPlaceholder	Все ресурсы	Полноценный блог
12	GitHub API	Пользователи + Репозитории	Профиль и список репозиториев
13	OpenWeather	Погода	Поиск по городу, прогноз
14	News API	Новости	Поиск по ключевым словам
15	YouTube API	Видео	Поиск, информация о канале
16	Spotify API	Треки	Поиск, плейлисты
17	Unsplash API	Изображения	Галерея, поиск, скачивание
Варианты 18-25 (Сложный уровень)
№	Тема	Требования
18	Интернет-магазин	Товары, корзина, оформление заказа
19	Блог с комментариями	Посты, комментарии, авторизация
20	Социальная сеть	Профили, посты, лента новостей
21	Система бронирования	Отели, номера, бронирование
22	Доска объявлений	Объявления, категории, поиск
23	Погодный сервис	Прогноз, история, графики
24	Курсы валют	Курсы, конвертер, графики
25	Агрегатор новостей	RSS лента, категории, поиск
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Приложение не работает или не отображает данные
3 (удовлетворительно)	Есть 2-3 страницы, базовый CSS
4 (хорошо)	Полный функционал, адаптивный дизайн
5 (отлично)	Профессиональный дизайн, обработка ошибок, поиск
5. Шпаргалка
5.1. Быстрый старт FastAPI + Jinja2
python
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles

app = FastAPI()
app.mount("/static", StaticFiles(directory="static"), name="static")
templates = Jinja2Templates(directory="templates")

@app.get("/")
async def home(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})
5.2. Получение данных из API
python
import requests

def get_data():
    response = requests.get("https://api.example.com/data")
    return response.json()
5.3. Передача данных в шаблон
python
@app.get("/page")
async def page(request: Request):
    data = get_data()
    return templates.TemplateResponse(
        "page.html",
        {"request": request, "data": data}
    )
5.4. Обработка ошибок API
python
try:
    data = get_data()
except requests.exceptions.RequestException:
    data = []
    error = "Не удалось загрузить данные"
Карточка студента
text
ПЗ 2.24. РАЗРАБОТКА ИНТЕРФЕЙСА ДЛЯ ОТОБРАЖЕНИЯ ДАННЫХ ИЗ API

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

API: _________________________________
Ресурсы: _____________________________

=== СТРАНИЦЫ ===

□ Главная страница (список)
□ Страница детального просмотра
□ Страница поиска/фильтрации
□ Страница пользователя/категории
□ Обработка ошибок (404, 500)
□ Адаптивный дизайн

=== КОМПОНЕНТЫ ===

□ Шаблон base.html (наследование)
□ CSS стили
□ Клиент API (services/)
□ Обработка ошибок API

=== ОТЧЁТ ===

Ссылка на репозиторий: _____________
Скриншоты страниц: _____________

Дата выполнения: _____________
