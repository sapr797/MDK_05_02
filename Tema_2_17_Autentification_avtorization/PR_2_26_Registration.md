# ПЗ 2.26. Реализация регистрации и входа пользователя (JWT)

**Тема:** Аутентификация и авторизация, JWT-токены, хэширование паролей, защита API

**Цель работы:**  
Научиться реализовывать полноценную систему аутентификации пользователей с использованием JWT-токенов: регистрация, вход, защита эндпоинтов, обновление токенов.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `fastapi`, `uvicorn`, `python-jose`, `passlib`, `python-multipart`

```bash
pip install fastapi uvicorn python-jose[cryptography] passlib[bcrypt] python-multipart
Главная мысль: JWT — это современный стандарт аутентификации. Он позволяет создавать stateless API, где серверу не нужно хранить сессии пользователей.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Компоненты системы аутентификации
Компонент	Назначение
Регистрация	Создание нового пользователя
Хэширование пароля	Безопасное хранение паролей (bcrypt)
JWT токен	Подписанный JSON с данными пользователя
Login эндпоинт	Проверка пароля, выдача токена
Protected эндпоинты	Требуют валидный токен
Refresh токен	Для обновления access токена
1.2. Схема работы
text
┌─────────┐          ┌─────────┐          ┌─────────┐
│ Клиент  │          │ Сервер  │          │  БД     │
└────┬────┘          └────┬────┘          └────┬────┘
     │                    │                    │
     │ POST /register     │                    │
     │ (name, email, pwd) │                    │
     │───────────────────►│                    │
     │                    │───────────────────►│
     │                    │  Сохраняем         │
     │                    │  (хэш пароля)      │
     │                    │◄───────────────────│
     │ 201 Created        │                    │
     │◄───────────────────│                    │
     │                    │                    │
     │ POST /login        │                    │
     │ (email, password)  │                    │
     │───────────────────►│                    │
     │                    │───────────────────►│
     │                    │  Проверяем         │
     │                    │◄───────────────────│
     │                    │                    │
     │ 200 OK             │                    │
     │ {access_token}     │                    │
     │◄───────────────────│                    │
     │                    │                    │
     │ GET /profile       │                    │
     │ Authorization:     │                    │
     │ Bearer {token}     │                    │
     │───────────────────►│                    │
     │                    │  Верификация       │
     │                    │  токена            │
     │                    │                    │
     │ 200 OK             │                    │
     │ {user_data}        │                    │
     │◄───────────────────│                    │
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая реализацию регистрации и входа с JWT.

Техническое задание (нулевой вариант)
Разработайте API для аутентификации пользователей с использованием JWT. API должно поддерживать:

Регистрацию новых пользователей (хэширование паролей)

Вход (логин) с выдачей JWT токена

Защищённый эндпоинт /profile (требует токен)

Защищённый эндпоинт /users/me для получения информации о текущем пользователе

Обновление токена (refresh)

Эталонная реализация
python
#!/usr/bin/env python3
"""
auth_api.py — API для аутентификации пользователей с JWT.
"""

import os
from datetime import datetime, timedelta
from typing import Optional, List, Dict
from fastapi import FastAPI, HTTPException, Depends, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, EmailStr, Field
from passlib.context import CryptContext
from jose import JWTError, jwt
import sqlite3
from contextlib import contextmanager

# ============================================================
# КОНФИГУРАЦИЯ
# ============================================================

SECRET_KEY = "your-super-secret-key-change-in-production-12345"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30
REFRESH_TOKEN_EXPIRE_DAYS = 7

# Настройка хэширования паролей
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Security
security = HTTPBearer()


# ============================================================
# МОДЕЛИ PYDANTIC
# ============================================================

class UserRegister(BaseModel):
    """Модель для регистрации пользователя."""
    name: str = Field(..., min_length=2, max_length=50)
    email: EmailStr
    password: str = Field(..., min_length=6)


class UserLogin(BaseModel):
    """Модель для входа пользователя."""
    email: EmailStr
    password: str


class UserResponse(BaseModel):
    """Модель ответа с данными пользователя."""
    id: int
    name: str
    email: str
    is_active: bool
    created_at: datetime


class TokenResponse(BaseModel):
    """Модель ответа с токенами."""
    access_token: str
    refresh_token: str
    token_type: str = "bearer"


class RefreshRequest(BaseModel):
    """Модель запроса на обновление токена."""
    refresh_token: str


# ============================================================
# РАБОТА С БАЗОЙ ДАННЫХ
# ============================================================

DATABASE_PATH = "users.db"


@contextmanager
def get_db():
    """Контекстный менеджер для подключения к БД."""
    conn = sqlite3.connect(DATABASE_PATH)
    conn.row_factory = sqlite3.Row
    try:
        yield conn
    finally:
        conn.close()


def init_database():
    """Инициализация базы данных."""
    with get_db() as conn:
        conn.execute("""
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT UNIQUE NOT NULL,
                password_hash TEXT NOT NULL,
                is_active BOOLEAN DEFAULT 1,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        
        conn.execute("""
            CREATE TABLE IF NOT EXISTS refresh_tokens (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER NOT NULL,
                token_jti TEXT UNIQUE NOT NULL,
                expires_at TIMESTAMP NOT NULL,
                revoked BOOLEAN DEFAULT 0,
                FOREIGN KEY (user_id) REFERENCES users(id)
            )
        """)
        
        conn.commit()


def get_user_by_email(email: str) -> Optional[Dict]:
    """Поиск пользователя по email."""
    with get_db() as conn:
        cursor = conn.execute(
            "SELECT * FROM users WHERE email = ?",
            (email.lower(),)
        )
        row = cursor.fetchone()
        return dict(row) if row else None


def get_user_by_id(user_id: int) -> Optional[Dict]:
    """Поиск пользователя по ID."""
    with get_db() as conn:
        cursor = conn.execute(
            "SELECT id, name, email, is_active, created_at FROM users WHERE id = ?",
            (user_id,)
        )
        row = cursor.fetchone()
        return dict(row) if row else None


def create_user(name: str, email: str, password_hash: str) -> int:
    """Создание нового пользователя."""
    with get_db() as conn:
        cursor = conn.execute(
            "INSERT INTO users (name, email, password_hash) VALUES (?, ?, ?)",
            (name, email.lower(), password_hash)
        )
        conn.commit()
        return cursor.lastrowid


def save_refresh_token(user_id: int, token_jti: str, expires_at: datetime) -> None:
    """Сохранение refresh токена."""
    with get_db() as conn:
        conn.execute(
            "INSERT INTO refresh_tokens (user_id, token_jti, expires_at) VALUES (?, ?, ?)",
            (user_id, token_jti, expires_at.isoformat())
        )
        conn.commit()


def revoke_refresh_token(token_jti: str) -> bool:
    """Отзыв refresh токена."""
    with get_db() as conn:
        cursor = conn.execute(
            "UPDATE refresh_tokens SET revoked = 1 WHERE token_jti = ?",
            (token_jti,)
        )
        conn.commit()
        return cursor.rowcount > 0


def is_refresh_token_valid(token_jti: str) -> bool:
    """Проверка валидности refresh токена."""
    with get_db() as conn:
        cursor = conn.execute(
            "SELECT revoked, expires_at FROM refresh_tokens WHERE token_jti = ?",
            (token_jti,)
        )
        row = cursor.fetchone()
        if not row:
            return False
        
        if row['revoked']:
            return False
        
        expires_at = datetime.fromisoformat(row['expires_at'])
        return expires_at > datetime.now()


# ============================================================
# JWT ФУНКЦИИ
# ============================================================

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    """Создание access токена."""
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES))
    to_encode.update({"exp": expire, "type": "access"})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)


def create_refresh_token(user_id: int, jti: str, expires_delta: Optional[timedelta] = None) -> str:
    """Создание refresh токена."""
    expire = datetime.utcnow() + (expires_delta or timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS))
    payload = {
        "sub": user_id,
        "jti": jti,
        "exp": expire,
        "type": "refresh"
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)


def verify_access_token(token: str) -> Optional[Dict]:
    """Верификация access токена."""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        
        if payload.get("type") != "access":
            return None
        
        return payload
    except JWTError:
        return None


def verify_refresh_token(token: str) -> Optional[Dict]:
    """Верификация refresh токена."""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        
        if payload.get("type") != "refresh":
            return None
        
        if not is_refresh_token_valid(payload.get("jti")):
            return None
        
        return payload
    except JWTError:
        return None


# ============================================================
# ЗАВИСИМОСТИ
# ============================================================

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> Dict:
    """Получение текущего пользователя из JWT токена."""
    token = credentials.credentials
    payload = verify_access_token(token)
    
    if payload is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный или истёкший токен",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    user_id = payload.get("sub")
    if not user_id:
        raise HTTPException(status_code=401, detail="Неверный токен")
    
    user = get_user_by_id(user_id)
    if not user:
        raise HTTPException(status_code=401, detail="Пользователь не найден")
    
    if not user.get("is_active"):
        raise HTTPException(status_code=403, detail="Пользователь заблокирован")
    
    return user


# ============================================================
# СОЗДАНИЕ ПРИЛОЖЕНИЯ
# ============================================================

app = FastAPI(
    title="Auth API",
    description="API для аутентификации пользователей с JWT",
    version="1.0.0"
)

# Инициализация БД при запуске
init_database()


# ============================================================
# ПУБЛИЧНЫЕ ЭНДПОИНТЫ
# ============================================================

@app.post("/register", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def register(user_data: UserRegister):
    """
    Регистрация нового пользователя.
    
    - Хранит пароль в хэшированном виде
    - Проверяет уникальность email
    """
    # Проверка существования пользователя
    existing = get_user_by_email(user_data.email)
    if existing:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Пользователь с таким email уже существует"
        )
    
    # Хэширование пароля
    password_hash = pwd_context.hash(user_data.password)
    
    # Создание пользователя
    user_id = create_user(
        name=user_data.name,
        email=user_data.email,
        password_hash=password_hash
    )
    
    # Получение созданного пользователя
    user = get_user_by_id(user_id)
    
    return UserResponse(
        id=user['id'],
        name=user['name'],
        email=user['email'],
        is_active=bool(user['is_active']),
        created_at=datetime.fromisoformat(user['created_at'])
    )


@app.post("/login", response_model=TokenResponse)
async def login(user_data: UserLogin):
    """
    Аутентификация пользователя.
    
    - Проверяет email и пароль
    - Выдаёт access и refresh токены
    """
    # Поиск пользователя
    user = get_user_by_email(user_data.email)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный email или пароль"
        )
    
    # Проверка пароля
    if not pwd_context.verify(user_data.password, user['password_hash']):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный email или пароль"
        )
    
    # Проверка активности
    if not user['is_active']:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Пользователь заблокирован"
        )
    
    # Создание токенов
    access_token = create_access_token(
        data={"sub": user['id'], "email": user['email']}
    )
    
    # Создание refresh токена
    import uuid
    jti = str(uuid.uuid4())
    refresh_token = create_refresh_token(user['id'], jti)
    
    # Сохранение refresh токена
    expires_at = datetime.utcnow() + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS)
    save_refresh_token(user['id'], jti, expires_at)
    
    return TokenResponse(
        access_token=access_token,
        refresh_token=refresh_token
    )


@app.post("/refresh", response_model=TokenResponse)
async def refresh_token(request: RefreshRequest):
    """
    Обновление access токена по refresh токену.
    """
    payload = verify_refresh_token(request.refresh_token)
    
    if payload is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный или истёкший refresh токен"
        )
    
    user_id = payload.get("sub")
    user = get_user_by_id(user_id)
    
    if not user or not user.get("is_active"):
        raise HTTPException(status_code=401, detail="Пользователь не найден или заблокирован")
    
    # Создание новых токенов
    access_token = create_access_token(
        data={"sub": user['id'], "email": user['email']}
    )
    
    import uuid
    new_jti = str(uuid.uuid4())
    refresh_token = create_refresh_token(user['id'], new_jti)
    
    expires_at = datetime.utcnow() + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS)
    save_refresh_token(user['id'], new_jti, expires_at)
    
    return TokenResponse(
        access_token=access_token,
        refresh_token=refresh_token
    )


@app.post("/logout")
async def logout(request: RefreshRequest, current_user: Dict = Depends(get_current_user)):
    """
    Выход из системы (отзыв refresh токена).
    """
    try:
        payload = jwt.decode(request.refresh_token, SECRET_KEY, algorithms=[ALGORITHM])
        jti = payload.get("jti")
        if jti:
            revoke_refresh_token(jti)
    except JWTError:
        pass
    
    return {"message": "Успешный выход"}


# ============================================================
# ЗАЩИЩЁННЫЕ ЭНДПОИНТЫ
# ============================================================

@app.get("/profile", response_model=UserResponse)
async def get_profile(current_user: Dict = Depends(get_current_user)):
    """
    Получение профиля текущего пользователя.
    Требует валидный JWT токен.
    """
    return UserResponse(
        id=current_user['id'],
        name=current_user['name'],
        email=current_user['email'],
        is_active=bool(current_user['is_active']),
        created_at=datetime.fromisoformat(current_user['created_at'])
    )


@app.get("/users/me")
async def get_current_user_info(current_user: Dict = Depends(get_current_user)):
    """
    Получение информации о текущем пользователе.
    """
    return {
        "id": current_user['id'],
        "name": current_user['name'],
        "email": current_user['email'],
        "is_active": current_user['is_active']
    }


@app.get("/protected")
async def protected_route(current_user: Dict = Depends(get_current_user)):
    """
    Пример защищённого маршрута.
    """
    return {
        "message": f"Привет, {current_user['name']}! Это защищённый маршрут.",
        "user_id": current_user['id']
    }


# ============================================================
# ДОПОЛНИТЕЛЬНЫЕ ЭНДПОИНТЫ (ДЛЯ АДМИНА)
# ============================================================

@app.get("/users", response_model=List[UserResponse])
async def get_all_users(current_user: Dict = Depends(get_current_user)):
    """
    Получение списка всех пользователей (только для админа).
    """
    # Здесь должна быть проверка роли
    # Для демонстрации просто возвращаем всех пользователей
    
    with get_db() as conn:
        cursor = conn.execute("SELECT id, name, email, is_active, created_at FROM users")
        users = []
        for row in cursor.fetchall():
            users.append(UserResponse(
                id=row['id'],
                name=row['name'],
                email=row['email'],
                is_active=bool(row['is_active']),
                created_at=datetime.fromisoformat(row['created_at'])
            ))
        return users


# ============================================================
# ЗАПУСК
# ============================================================

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        "auth_api:app",
        host="127.0.0.1",
        port=8000,
        reload=True
    )
Тестирование API
Тест 1: Регистрация
bash
curl -X POST http://localhost:8000/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Иван Петров",
    "email": "ivan@example.com",
    "password": "secret123"
  }'
Ответ:

json
{
    "id": 1,
    "name": "Иван Петров",
    "email": "ivan@example.com",
    "is_active": true,
    "created_at": "2024-01-15T14:30:25.123456"
}
Тест 2: Логин
bash
curl -X POST http://localhost:8000/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "ivan@example.com",
    "password": "secret123"
  }'
Ответ:

json
{
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
    "token_type": "bearer"
}
Тест 3: Получение профиля (защищённый маршрут)
bash
TOKEN="eyJhbGciOiJIUzI1NiIs..."
curl -X GET http://localhost:8000/profile \
  -H "Authorization: Bearer $TOKEN"
Ответ:

json
{
    "id": 1,
    "name": "Иван Петров",
    "email": "ivan@example.com",
    "is_active": true,
    "created_at": "2024-01-15T14:30:25.123456"
}
Тест 4: Обновление токена
bash
REFRESH_TOKEN="eyJhbGciOiJIUzI1NiIs..."
curl -X POST http://localhost:8000/refresh \
  -H "Content-Type: application/json" \
  -d "{\"refresh_token\": \"$REFRESH_TOKEN\"}"
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (регистрация, логин, защита 1-2 эндпоинтов)

Варианты 9-17: средний (+ refresh токены, роли)

Варианты 18-25: сложный (+ email верификация, сброс пароля, 2FA)

Варианты 1-8 (Базовый уровень)
№	Тема API	Дополнительные эндпоинты
1	Заметки	GET /notes, POST /notes
2	Контакты	GET /contacts, POST /contacts
3	Задачи	GET /todos, POST /todos
4	Книги	GET /books, POST /books
5	Фильмы	GET /movies, POST /movies
6	Студенты	GET /students, POST /students
7	События	GET /events, POST /events
8	Расходы	GET /expenses, POST /expenses
Варианты 9-17 (Средний уровень)
№	Тема	Дополнительные требования
9	Блог	Посты, комментарии (только автор может редактировать)
10	Магазин	Товары, корзина (только владелец корзины)
11	Задачи	Задачи, проекты (роли: user, manager)
12	Библиотека	Книги, бронирования (роли: читатель, библиотекарь)
13	CRM	Клиенты, сделки (роли: менеджер, директор)
14	Календарь	События, приглашения (доступ по email)
15	Чат	Сообщения, комнаты (только участники)
16	Форум	Темы, сообщения (роли: участник, модератор)
17	Админ-панель	Управление пользователями (роль: admin)
Варианты 18-25 (Сложный уровень)
№	Тема	Требования
18	Социальная сеть	Профили, друзья, посты, лайки
19	Интернет-магазин	Корзина, заказы, платёжная система
20	Система бронирования	Отели, номера, бронирование, отзывы
21	Образовательная платформа	Курсы, уроки, прогресс студентов
22	Система голосования	Выборы, кандидаты, голоса (один голос на пользователя)
23	Документооборот	Документы, подписи, версионирование
24	Платформа доставки	Заказы, курьеры, трекинг
25	Крипто-биржа	Кошельки, транзакции, ордера
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Регистрация/логин не работают
3 (удовлетворительно)	Регистрация и логин работают
4 (хорошо)	+ Защищённые эндпоинты, хэширование паролей
5 (отлично)	+ Refresh токены, роли, полноценная авторизация
5. Шпаргалка
python
# === ХЭШИРОВАНИЕ ПАРОЛЯ ===
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Хэширование
password_hash = pwd_context.hash("secret123")

# Проверка
is_valid = pwd_context.verify("secret123", password_hash)

# === СОЗДАНИЕ JWT ===
from jose import jwt
from datetime import datetime, timedelta

payload = {"sub": user_id, "exp": datetime.utcnow() + timedelta(minutes=30)}
token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")

# === ВЕРИФИКАЦИЯ JWT ===
try:
    payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
except jwt.ExpiredSignatureError:
    print("Токен истёк")
except jwt.InvalidTokenError:
    print("Неверный токен")

# === ЗАВИСИМОСТЬ FASTAPI ===
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer

security = HTTPBearer()

async def get_current_user(credentials = Depends(security)):
    token = credentials.credentials
    payload = verify_token(token)
    return payload
Карточка студента
text
ПЗ 2.26. РЕАЛИЗАЦИЯ РЕГИСТРАЦИИ И ВХОДА ПОЛЬЗОВАТЕЛЯ (JWT)

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Тема приложения: _________________________________

=== РЕАЛИЗОВАННЫЕ ФУНКЦИИ ===

□ Регистрация пользователя
□ Хэширование паролей (bcrypt)
□ Логин (выдача JWT)
□ Защищённые эндпоинты
□ Получение профиля (/profile)
□ Refresh токены
□ Роли пользователей (admin/user)
□ Email верификация
□ Сброс пароля

=== ЗАЩИЩЁННЫЕ ЭНДПОИНТЫ ===

1. _________________________________
2. _________________________________
3. _________________________________

=== ОТЧЁТ ===

Ссылка на репозиторий: _____________
Скриншоты тестирования: _____________

Дата выполнения: _____________
