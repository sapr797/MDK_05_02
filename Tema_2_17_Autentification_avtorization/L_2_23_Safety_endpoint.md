# Тема 2.23. Защита API-эндпоинтов. Middleware, зависимости

**Цель лекции:**  
Изучить методы защиты API-эндпоинтов в FastAPI, освоить создание middleware для глобальной обработки запросов, использовать зависимости (Dependency Injection) для проверки прав доступа, реализовать защиту от основных веб-угроз.

> Главная мысль: **Безопасность API — это не опция, а обязательное требование. Middleware и зависимости — главные инструменты для её обеспечения.**

---

## Содержание

1. [Введение в защиту API](#1-введение-в-защиту-api)
2. [Middleware в FastAPI](#2-middleware-в-fastapi)
3. [Зависимости (Dependency Injection)](#3-зависимости-dependency-injection)
4. [Защита от основных угроз](#4-защита-от-основных-угроз)
5. [Практические примеры](#5-практические-примеры)
6. [Контрольные вопросы](#6-контрольные-вопросы)
7. [Практическое задание](#7-практическое-задание)
8. [Шпаргалка](#8-шпаргалка)

---

## 1. Введение в защиту API

### 1.1. Основные угрозы для API

| Угроза | Описание | Последствия |
|--------|----------|-------------|
| **Неавторизованный доступ** | Доступ без токена или логина | Утечка данных |
| **Подделка токена** | Использование поддельного JWT | Компрометация аккаунта |
| **Replay Attack** | Повторная отправка запроса | Дублирование операций |
| **Brute Force** | Перебор паролей | Взлом аккаунтов |
| **DDoS** | Перегрузка сервера | Отказ в обслуживании |
| **SQL Injection** | Внедрение SQL кода | Утечка/удаление данных |
| **XSS** | Внедрение скриптов | Кража сессий |

### 1.2. Уровни защиты API
┌─────────────────────────────────────────────────────────────────┐
│ УРОВЕНЬ 1: ТРАНСПОРТ │
│ HTTPS, CORS │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ УРОВЕНЬ 2: МIDDLEWARE │
│ Лимиты запросов, Логирование, Сжатие │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ УРОВЕНЬ 3: АУТЕНТИФИКАЦИЯ │
│ JWT, API Key, Basic Auth │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ УРОВЕНЬ 4: АВТОРИЗАЦИЯ │
│ Роли, Права, Доступы │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ УРОВЕНЬ 5: ВАЛИДАЦИЯ │
│ Проверка входных данных, Санитизация │
└─────────────────────────────────────────────────────────────────┘

text

---

## 2. Middleware в FastAPI

### 2.1. Что такое Middleware

**Middleware** — это функция, которая выполняется для каждого запроса до того, как он попадёт в эндпоинт, и для каждого ответа после.
┌─────────┐ ┌──────────────┐ ┌──────────────┐ ┌─────────┐
│ Запрос │───►│ Middleware │───►│ Endpoint │───►│ Ответ │
│ │ │ (обработка) │ │ (обработка) │ │ │
└─────────┘ └──────────────┘ └──────────────┘ └─────────┘
│ │
│ Middleware │
└───────────────(обработка)─────────────┘

text

### 2.2. Базовое Middleware

```python
from fastapi import FastAPI, Request
from starlette.middleware.base import BaseHTTPMiddleware
import time

app = FastAPI()

# Простое middleware через декоратор
@app.middleware("http")
async def log_requests(request: Request, call_next):
    start_time = time.time()
    
    print(f"📥 Запрос: {request.method} {request.url.path}")
    
    response = await call_next(request)
    
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    
    print(f"📤 Ответ: {response.status_code} ({process_time:.3f}s)")
    
    return response


@app.get("/")
async def root():
    return {"message": "Hello World"}
2.3. Middleware через класс
python
from fastapi import FastAPI, Request
from starlette.middleware.base import BaseHTTPMiddleware
from typing import Callable
import time

class LoggingMiddleware(BaseHTTPMiddleware):
    """Middleware для логирования запросов."""
    
    async def dispatch(self, request: Request, call_next: Callable):
        # Действия ДО обработки запроса
        start_time = time.time()
        print(f"[{request.method}] {request.url.path}")
        
        # Передача запроса дальше
        response = await call_next(request)
        
        # Действия ПОСЛЕ обработки
        process_time = time.time() - start_time
        response.headers["X-Response-Time"] = str(process_time)
        
        return response


class AuthMiddleware(BaseHTTPMiddleware):
    """Middleware для проверки аутентификации."""
    
    async def dispatch(self, request: Request, call_next: Callable):
        # Список публичных маршрутов (не требуют токена)
        public_paths = ['/', '/docs', '/redoc', '/openapi.json', '/login', '/register']
        
        if request.url.path in public_paths:
            return await call_next(request)
        
        # Получаем токен из заголовка
        auth_header = request.headers.get('Authorization')
        
        if not auth_header or not auth_header.startswith('Bearer '):
            from fastapi.responses import JSONResponse
            return JSONResponse(
                status_code=401,
                content={"detail": "Требуется авторизация"}
            )
        
        token = auth_header.replace('Bearer ', '')
        
        # Здесь должна быть верификация токена
        # request.state.user = verify_token(token)
        
        return await call_next(request)


# Подключение middleware
app.add_middleware(LoggingMiddleware)
app.add_middleware(AuthMiddleware)
2.4. Встроенные middleware
python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from fastapi.middleware.gzip import GZipMiddleware

app = FastAPI()

# CORS (Cross-Origin Resource Sharing)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com", "http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
)

# Перенаправление HTTP → HTTPS
app.add_middleware(HTTPSRedirectMiddleware)

# Ограничение доверенных хостов
app.add_middleware(
    TrustedHostMiddleware,
    allowed_hosts=["example.com", "*.example.com"]
)

# GZip сжатие ответов
app.add_middleware(GZipMiddleware, minimum_size=1000)
3. Зависимости (Dependency Injection)
3.1. Что такое зависимости
Зависимости (Dependencies) — это механизм внедрения общих компонентов в эндпоинты.

python
from fastapi import FastAPI, Depends, Header, HTTPException

app = FastAPI()

# Простая зависимость
def common_parameters(page: int = 1, limit: int = 10):
    return {"page": page, "limit": limit}

@app.get("/items")
def get_items(params: dict = Depends(common_parameters)):
    return params

# Зависимость с логикой
async def get_current_user(token: str = Header(...)):
    if token != "secret":
        raise HTTPException(status_code=401, detail="Неверный токен")
    return {"id": 1, "name": "Иван"}

@app.get("/profile")
def get_profile(user: dict = Depends(get_current_user)):
    return user
3.2. Зависимости с уровнями
python
from fastapi import FastAPI, Depends, HTTPException, status
from typing import Annotated

app = FastAPI()

# Уровень 1: Проверка токена
async def get_token(authorization: str = Header(...)) -> str:
    if not authorization.startswith('Bearer '):
        raise HTTPException(status_code=401, detail="Неверный формат токена")
    return authorization.replace('Bearer ', '')

# Уровень 2: Проверка пользователя
async def get_current_user(token: str = Depends(get_token)) -> dict:
    # Здесь верификация JWT
    if token == "invalid":
        raise HTTPException(status_code=401, detail="Неверный токен")
    return {"id": 1, "name": "Иван", "role": "user"}

# Уровень 3: Проверка прав
async def get_admin_user(user: dict = Depends(get_current_user)) -> dict:
    if user.get('role') != 'admin':
        raise HTTPException(status_code=403, detail="Недостаточно прав")
    return user

# Использование
@app.get("/profile")
def get_profile(user: dict = Depends(get_current_user)):
    return user

@app.get("/admin")
def admin_panel(admin: dict = Depends(get_admin_user)):
    return {"message": f"Добро пожаловать, {admin['name']}!"}

# Альтернативный синтаксис с Annotated (Python 3.9+)
from typing import Annotated

@app.get("/me")
def get_me(current_user: Annotated[dict, Depends(get_current_user)]):
    return current_user
3.3. Классы как зависимости
python
from fastapi import FastAPI, Depends

app = FastAPI()

class Pagination:
    def __init__(self, page: int = 1, limit: int = 10):
        self.page = page
        self.limit = limit
        self.offset = (page - 1) * limit
    
    def __call__(self):
        return {"page": self.page, "limit": self.limit, "offset": self.offset}

class AuthService:
    def __init__(self, secret_key: str = "default_secret"):
        self.secret_key = secret_key
    
    def verify_token(self, token: str) -> dict:
        # Верификация токена
        return {"id": 1, "name": "Иван"}

# Создаём экземпляры
pagination = Pagination()
auth_service = AuthService()

@app.get("/items")
def get_items(pagination_params: dict = Depends(pagination)):
    return pagination_params

@app.get("/protected")
def protected_route(user: dict = Depends(auth_service.verify_token)):
    return user
3.4. Зависимости с yield (для ресурсов)
python
from fastapi import FastAPI, Depends
from contextlib import contextmanager

app = FastAPI()

# Зависимость с управлением ресурсами
def get_db():
    db = DatabaseConnection()
    try:
        yield db
    finally:
        db.close()

@app.get("/users")
def get_users(db = Depends(get_db)):
    return db.query("SELECT * FROM users")

# Использование contextmanager
@contextmanager
def get_file(filename: str):
    file = open(filename, 'r')
    try:
        yield file
    finally:
        file.close()

def read_config(file = Depends(lambda: get_file("config.json"))):
    return json.load(file)
4. Защита от основных угроз
4.1. Rate Limiting (ограничение запросов)
python
from fastapi import FastAPI, Request, HTTPException
from starlette.middleware.base import BaseHTTPMiddleware
from collections import defaultdict
import time

class RateLimitMiddleware(BaseHTTPMiddleware):
    """Middleware для ограничения количества запросов."""
    
    def __init__(self, app, requests_per_minute: int = 60):
        super().__init__(app)
        self.requests_per_minute = requests_per_minute
        self.requests: defaultdict = defaultdict(list)
    
    async def dispatch(self, request: Request, call_next):
        # Определяем клиента (IP или API ключ)
        client_ip = request.client.host
        
        # Очищаем старые записи
        now = time.time()
        self.requests[client_ip] = [
            req_time for req_time in self.requests[client_ip]
            if now - req_time < 60
        ]
        
        # Проверяем лимит
        if len(self.requests[client_ip]) >= self.requests_per_minute:
            from fastapi.responses import JSONResponse
            return JSONResponse(
                status_code=429,
                content={"detail": "Too Many Requests. Please try again later."}
            )
        
        # Добавляем текущий запрос
        self.requests[client_ip].append(now)
        
        return await call_next(request)


# Более гибкая версия с декоратором
from functools import wraps

def rate_limit(limit: int, window: int = 60):
    """Декоратор для ограничения запросов к конкретному эндпоинту."""
    requests = defaultdict(list)
    
    def decorator(func):
        @wraps(func)
        async def wrapper(request: Request, *args, **kwargs):
            client_ip = request.client.host
            now = time.time()
            
            # Очистка
            requests[client_ip] = [
                t for t in requests[client_ip] if now - t < window
            ]
            
            if len(requests[client_ip]) >= limit:
                raise HTTPException(status_code=429, detail="Rate limit exceeded")
            
            requests[client_ip].append(now)
            return await func(request, *args, **kwargs)
        
        return wrapper
    return decorator


app.add_middleware(RateLimitMiddleware, requests_per_minute=60)

@app.get("/api/data")
@rate_limit(limit=10, window=60)  # 10 запросов в минуту
async def get_data(request: Request):
    return {"data": "secret data"}
4.2. Защита от Brute Force
python
from fastapi import FastAPI, HTTPException, Depends
from datetime import datetime, timedelta
from collections import defaultdict

app = FastAPI()

# Хранилище попыток входа
login_attempts: defaultdict = defaultdict(list)
MAX_ATTEMPTS = 5
LOCKOUT_MINUTES = 15

def check_brute_force(ip: str, email: str):
    """Проверка на брутфорс."""
    now = datetime.now()
    
    # Очистка старых попыток
    for key in list(login_attempts.keys()):
        if now - login_attempts[key][-1] > timedelta(minutes=LOCKOUT_MINUTES):
            del login_attempts[key]
    
    # Проверка по IP
    ip_key = f"ip:{ip}"
    if ip_key in login_attempts:
        if len(login_attempts[ip_key]) >= MAX_ATTEMPTS:
            raise HTTPException(status_code=429, detail="Слишком много попыток. Попробуйте позже.")
    
    # Проверка по email
    email_key = f"email:{email}"
    if email_key in login_attempts:
        if len(login_attempts[email_key]) >= MAX_ATTEMPTS:
            raise HTTPException(status_code=429, detail="Аккаунт временно заблокирован")

def record_failed_attempt(ip: str, email: str):
    """Запись неудачной попытки."""
    now = datetime.now()
    login_attempts[f"ip:{ip}"].append(now)
    login_attempts[f"email:{email}"].append(now)

@app.post("/login")
async def login(request: Request, email: str, password: str):
    client_ip = request.client.host
    
    # Проверка на брутфорс
    check_brute_force(client_ip, email)
    
    # Проверка логина (упрощённо)
    if email == "admin@example.com" and password == "password":
        return {"token": "fake-jwt-token"}
    
    # Неудачная попытка
    record_failed_attempt(client_ip, email)
    raise HTTPException(status_code=401, detail="Неверный email или пароль")
4.3. Валидация входных данных
python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field, validator
import re

app = FastAPI()

class UserCreate(BaseModel):
    name: str = Field(..., min_length=2, max_length=50)
    email: str
    password: str = Field(..., min_length=8)
    
    @validator('email')
    def validate_email(cls, v):
        # Проверка на XSS в email
        if re.search(r'<[^>]*>', v):
            raise ValueError('Недопустимые символы в email')
        
        if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', v):
            raise ValueError('Неверный формат email')
        return v.lower()
    
    @validator('password')
    def validate_password(cls, v):
        # Проверка на сложность пароля
        if not re.search(r'[A-Z]', v):
            raise ValueError('Пароль должен содержать заглавную букву')
        if not re.search(r'\d', v):
            raise ValueError('Пароль должен содержать цифру')
        return v


@app.post("/users")
def create_user(user: UserCreate):
    # Данные уже валидированы Pydantic
    return {"message": "User created"}
4.4. Защита от SQL Injection
python
from fastapi import FastAPI, HTTPException
from sqlalchemy import text

app = FastAPI()

# ПЛОХО: уязвимо для SQL Injection
@app.get("/users/bad/{user_id}")
def get_user_bad(user_id: str):
    # Никогда так не делайте!
    query = f"SELECT * FROM users WHERE id = {user_id}"
    # results = db.execute(query)
    return {"warning": "This is vulnerable to SQL Injection!"}

# ХОРОШО: использование параметризованных запросов
@app.get("/users/good/{user_id}")
def get_user_good(user_id: int):
    # Использование параметров
    query = text("SELECT * FROM users WHERE id = :user_id")
    # results = db.execute(query, {"user_id": user_id})
    return {"message": "Safe query"}

# ХОРОШО: с ORM (SQLAlchemy)
# user = db.query(User).filter(User.id == user_id).first()
4.5. Защита от DoS через большие запросы
python
from fastapi import FastAPI, Request, HTTPException
from starlette.middleware.base import BaseHTTPMiddleware

class MaxBodySizeMiddleware(BaseHTTPMiddleware):
    """Middleware для ограничения размера тела запроса."""
    
    def __init__(self, app, max_size: int = 1_000_000):  # 1 MB
        super().__init__(app)
        self.max_size = max_size
    
    async def dispatch(self, request: Request, call_next):
        content_length = request.headers.get('content-length')
        
        if content_length and int(content_length) > self.max_size:
            from fastapi.responses import JSONResponse
            return JSONResponse(
                status_code=413,
                content={"detail": f"Request too large. Max size: {self.max_size} bytes"}
            )
        
        return await call_next(request)


app.add_middleware(MaxBodySizeMiddleware, max_size=1_000_000)
5. Практические примеры
5.1. Полная защита API (комплексный пример)
python
# secure_api.py
from fastapi import FastAPI, Request, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from fastapi.middleware.cors import CORSMiddleware
from starlette.middleware.base import BaseHTTPMiddleware
from pydantic import BaseModel, EmailStr, Field
from typing import Optional, List
from datetime import datetime, timedelta
from collections import defaultdict
import time
import jwt
import re

# ============================================================
# КОНФИГУРАЦИЯ
# ============================================================

SECRET_KEY = "your-secret-key-here"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30
MAX_REQUESTS_PER_MINUTE = 60
MAX_BODY_SIZE = 1_000_000  # 1 MB

# ============================================================
# MIDDLEWARE
# ============================================================

class SecurityMiddleware(BaseHTTPMiddleware):
    """Комплексное middleware для безопасности."""
    
    def __init__(self, app):
        super().__init__(app)
        self.request_counts = defaultdict(list)
    
    async def dispatch(self, request: Request, call_next):
        # 1. Проверка размера запроса
        content_length = request.headers.get('content-length')
        if content_length and int(content_length) > MAX_BODY_SIZE:
            from fastapi.responses import JSONResponse
            return JSONResponse(
                status_code=413,
                content={"detail": "Request too large"}
            )
        
        # 2. Rate limiting
        client_ip = request.client.host
        now = time.time()
        
        # Очистка старых записей
        self.request_counts[client_ip] = [
            t for t in self.request_counts[client_ip] if now - t < 60
        ]
        
        if len(self.request_counts[client_ip]) >= MAX_REQUESTS_PER_MINUTE:
            return JSONResponse(
                status_code=429,
                content={"detail": "Too many requests"}
            )
        
        self.request_counts[client_ip].append(now)
        
        # 3. Защита от подозрительных User-Agent
        user_agent = request.headers.get('user-agent', '')
        suspicious = ['curl', 'wget', 'python-requests', 'Go-http-client']
        
        for s in suspicious:
            if s.lower() in user_agent.lower():
                return JSONResponse(
                    status_code=403,
                    content={"detail": "Forbidden"}
                )
        
        # 4. Добавление security headers
        response = await call_next(request)
        response.headers['X-Content-Type-Options'] = 'nosniff'
        response.headers['X-Frame-Options'] = 'DENY'
        response.headers['X-XSS-Protection'] = '1; mode=block'
        response.headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
        
        return response


# ============================================================
# ЗАВИСИМОСТИ
# ============================================================

security = HTTPBearer()

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)


def verify_token(token: str) -> dict:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")


async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> dict:
    token = credentials.credentials
    payload = verify_token(token)
    
    user_id = payload.get("sub")
    if not user_id:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    return {"id": user_id, "role": payload.get("role", "user")}


async def get_admin_user(current_user: dict = Depends(get_current_user)) -> dict:
    if current_user.get("role") != "admin":
        raise HTTPException(status_code=403, detail="Admin access required")
    return current_user


# ============================================================
# МОДЕЛИ
# ============================================================

class UserLogin(BaseModel):
    email: EmailStr
    password: str = Field(..., min_length=1)


class TokenResponse(BaseModel):
    access_token: str
    token_type: str


# ============================================================
# ПРИЛОЖЕНИЕ
# ============================================================

app = FastAPI(title="Secure API")

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourdomain.com"],
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
)

# Security middleware
app.add_middleware(SecurityMiddleware)


# ============================================================
# ЭНДПОИНТЫ
# ============================================================

@app.post("/login", response_model=TokenResponse)
async def login(user: UserLogin):
    """Аутентификация пользователя."""
    # Проверка логина (упрощённо)
    if user.email == "admin@example.com" and user.password == "password":
        token = create_access_token({"sub": "1", "role": "admin"})
        return TokenResponse(access_token=token, token_type="bearer")
    
    if user.email == "user@example.com" and user.password == "password":
        token = create_access_token({"sub": "2", "role": "user"})
        return TokenResponse(access_token=token, token_type="bearer")
    
    raise HTTPException(status_code=401, detail="Invalid credentials")


@app.get("/protected")
async def protected_route(current_user: dict = Depends(get_current_user)):
    """Защищённый маршрут (требует токен)."""
    return {
        "message": "You have access!",
        "user_id": current_user["id"],
        "role": current_user["role"]
    }


@app.get("/admin")
async def admin_route(admin: dict = Depends(get_admin_user)):
    """Административный маршрут (требует права admin)."""
    return {"message": f"Welcome, admin {admin['id']}!"}


@app.get("/public")
async def public_route():
    """Публичный маршрут."""
    return {"message": "This is public"}


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000)
6. Контрольные вопросы
Что такое middleware и как его добавить в FastAPI?

В чём разница между middleware и зависимостью (Dependency)?

Как реализовать rate limiting в FastAPI?

Какие встроенные middleware есть в FastAPI?

Как создать зависимость с несколькими уровнями (Depends chain)?

Как защитить API от SQL Injection?

Какие security headers нужно добавлять в ответ?

Как реализовать проверку прав доступа (роли пользователей)?

Что такое CORS и как его настроить?

Как защитить API от брутфорса?

7. Практическое задание
Задание 1 (базовое)
Создайте middleware для логирования всех запросов с временем выполнения.

Задание 2 (среднее)
Реализуйте систему ролей (user, moderator, admin) с проверкой прав через зависимости.

Задание 3 (сложное)
Создайте полностью защищённое API с:

Rate limiting

JWT аутентификацией

Ролевой моделью

Валидацией входных данных

Security headers

Логированием

8. Шпаргалка
python
# === MIDDLEWARE ===
@app.middleware("http")
async def my_middleware(request: Request, call_next):
    # до запроса
    response = await call_next(request)
    # после запроса
    return response

# === ЗАВИСИМОСТИ ===
async def dependency():
    return {"data": "value"}

@app.get("/")
def endpoint(dep: dict = Depends(dependency)):
    return dep

# === ЦЕПОЧКА ЗАВИСИМОСТЕЙ ===
async def dep1(): return 1
async def dep2(value: int = Depends(dep1)): return value * 2

# === АННОТАЦИИ ===
from typing import Annotated
def endpoint(user: Annotated[dict, Depends(get_user)]): pass

# === SECURITY HEADERS ===
response.headers['X-Content-Type-Options'] = 'nosniff'
response.headers['X-Frame-Options'] = 'DENY'
response.headers['X-XSS-Protection'] = '1; mode=block'

# === CORS ===
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)
Итог лекции
Вы сегодня:

✅ Изучили middleware и их применение для защиты API

✅ Освоили механизм зависимостей (Dependency Injection)

✅ Реализовали rate limiting, защиту от брутфорса

✅ Настроили CORS, security headers

✅ Создали комплексную систему защиты API

Теперь ваши API защищены от основных веб-угроз!

