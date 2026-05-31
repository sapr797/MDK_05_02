# Тема 2.22. Аутентификация в API. JWT-токены, работа с токенами

**Цель лекции:**  
Изучить механизмы аутентификации в API, освоить работу с JWT-токенами (JSON Web Tokens), научиться создавать, верифицировать и обновлять токены, реализовать защиту API-эндпоинтов.

> Главная мысль: **Аутентификация — это проверка "кто ты?", авторизация — "что тебе можно?". JWT — безопасный способ передавать эти данные.**

---

## Содержание

1. [Введение в аутентификацию](#1-введение-в-аутентификацию)
2. [JWT: JSON Web Tokens](#2-jwt-json-web-tokens)
3. [Структура JWT](#3-структура-jwt)
4. [Работа с JWT в Python](#4-работа-с-jwt-в-python)
5. [Аутентификация в FastAPI](#5-аутентификация-в-fastapi)
6. [Refresh-токены](#6-refresh-токены)
7. [Практические примеры](#7-практические-примеры)
8. [Контрольные вопросы](#8-контрольные-вопросы)
9. [Практическое задание](#9-практическое-задание)
10. [Шпаргалка](#10-шпаргалка)

---

## 1. Введение в аутентификацию

### 1.1. Аутентификация vs Авторизация

| Термин | Описание | Вопрос | Пример |
|--------|----------|--------|--------|
| **Аутентификация** | Проверка подлинности пользователя | "Кто ты?" | Ввод логина и пароля |
| **Авторизация** | Проверка прав доступа | "Что тебе можно?" | Доступ к админ-панели |

### 1.2. Способы аутентификации в API

| Способ | Описание | Плюсы | Минусы |
|--------|----------|-------|--------|
| **Basic Auth** | Логин:пароль в заголовке | Простота | Небезопасен без HTTPS |
| **API Key** | Уникальный ключ | Простота | Нет срока действия |
| **Bearer Token** | Токен в заголовке | Гибкость | Требует управления |
| **JWT** | JSON Web Token | Самодостаточен | Нельзя отозвать |
| **OAuth 2.0** | Делегированный доступ | Безопасность | Сложность |

### 1.3. Схема работы с JWT
┌─────────┐ ┌─────────┐ ┌─────────┐
│ Клиент │ │ Сервер │ │ БД │
└────┬────┘ └────┬────┘ └────┬────┘
│ │ │
│ POST /login │ │
│ (email, password) │ │
│───────────────────►│ │
│ │───────────────────►│
│ │ Проверка │
│ │◄───────────────────│
│ │ │
│ 200 OK │ │
│ {token: "..."} │ │
│◄───────────────────│ │
│ │ │
│ GET /profile │ │
│ Authorization: │ │
│ Bearer {token} │ │
│───────────────────►│ │
│ │ Верификация │
│ │ токена │
│ │ │
│ 200 OK │ │
│ {user: {...}} │ │
│◄───────────────────│ │
│ │ │

text

---

## 2. JWT: JSON Web Tokens

### 2.1. Что такое JWT

**JWT (JSON Web Token)** — открытый стандарт (RFC 7519) для создания токенов доступа.

**Характеристики JWT:**
- Самодостаточность — содержит всю информацию о пользователе
- Цифровая подпись — гарантирует подлинность
- Компактность — малый размер
- Универсальность — используется в REST API, WebSockets, GraphQL

### 2.2. Установка библиотек

```bash
# Основная библиотека для JWT
pip install pyjwt

# Дополнительно для FastAPI
pip install fastapi uvicorn python-multipart
2.3. Простейший пример
python
import jwt
from datetime import datetime, timedelta

# Секретный ключ (в реальном приложении хранить в .env)
SECRET_KEY = "your-secret-key-keep-it-safe"
ALGORITHM = "HS256"

# Данные для токена (payload)
payload = {
    "user_id": 123,
    "username": "ivan",
    "email": "ivan@example.com",
    "exp": datetime.utcnow() + timedelta(hours=1)  # срок действия 1 час
}

# Создание токена
token = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
print(f"Токен: {token}")

# Декодирование токена
try:
    decoded = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    print(f"Декодировано: {decoded}")
except jwt.ExpiredSignatureError:
    print("Токен истёк")
except jwt.InvalidTokenError:
    print("Неверный токен")
3. Структура JWT
3.1. Три части JWT
JWT состоит из трёх частей, разделённых точками:

text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6I
伊Ван",ImlhdCI6MTUxNjIzOTAyMn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
Три части:

Header (заголовок) — алгоритм подписи и тип токена

Payload (полезная нагрузка) — данные (claims)

Signature (подпись) — подпись для верификации

3.2. Header
json
{
    "alg": "HS256",
    "typ": "JWT"
}
3.3. Payload (Claims)
Зарезервированные claims:

Claim	Название	Описание
iss	Issuer	Кто выпустил токен
sub	Subject	Тема (обычно user_id)
aud	Audience	Получатель токена
exp	Expiration Time	Время истечения (Unix timestamp)
nbf	Not Before	Время, с которого токен действителен
iat	Issued At	Время выпуска
jti	JWT ID	Уникальный идентификатор
Пользовательские claims:

json
{
    "user_id": 123,
    "username": "ivan",
    "email": "ivan@example.com",
    "role": "admin",
    "exp": 1704067200
}
3.4. Signature
python
# Подпись создаётся так:
signature = HMACSHA256(
    base64UrlEncode(header) + "." + base64UrlEncode(payload),
    secret_key
)
4. Работа с JWT в Python
4.1. Создание и верификация токенов
python
import jwt
from datetime import datetime, timedelta
from typing import Dict, Optional

class JWTManager:
    """Менеджер для работы с JWT-токенами."""
    
    def __init__(self, secret_key: str, algorithm: str = "HS256"):
        self.secret_key = secret_key
        self.algorithm = algorithm
    
    def create_token(self, data: Dict, expires_in: int = 3600) -> str:
        """
        Создаёт JWT-токен.
        
        Args:
            data: Данные для включения в токен
            expires_in: Время жизни в секундах
        
        Returns:
            JWT-токен
        """
        payload = data.copy()
        payload['exp'] = datetime.utcnow() + timedelta(seconds=expires_in)
        payload['iat'] = datetime.utcnow()
        
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)
    
    def verify_token(self, token: str) -> Optional[Dict]:
        """
        Проверяет и декодирует JWT-токен.
        
        Args:
            token: JWT-токен
        
        Returns:
            Декодированные данные или None при ошибке
        """
        try:
            payload = jwt.decode(
                token,
                self.secret_key,
                algorithms=[self.algorithm]
            )
            return payload
        except jwt.ExpiredSignatureError:
            print("Токен истёк")
            return None
        except jwt.InvalidTokenError as e:
            print(f"Неверный токен: {e}")
            return None
    
    def refresh_token(self, token: str, expires_in: int = 3600) -> Optional[str]:
        """
        Обновляет токен (создаёт новый с теми же данными).
        
        Args:
            token: Старый токен
            expires_in: Новое время жизни
        
        Returns:
            Новый токен или None
        """
        payload = self.verify_token(token)
        if payload:
            # Удаляем старые временные метки
            payload.pop('exp', None)
            payload.pop('iat', None)
            return self.create_token(payload, expires_in)
        return None


# Использование
jwt_manager = JWTManager(secret_key="my-super-secret-key")

# Создание токена
token = jwt_manager.create_token(
    data={"user_id": 1, "username": "ivan", "role": "admin"},
    expires_in=3600
)
print(f"Токен: {token}")

# Верификация
decoded = jwt_manager.verify_token(token)
print(f"Декодировано: {decoded}")

# Обновление
new_token = jwt_manager.refresh_token(token)
print(f"Новый токен: {new_token}")
4.2. Использование RS256 (асимметричное шифрование)
python
import jwt
from cryptography.hazmat.primitives import serialization

# Генерация ключей (один раз)
# openssl genrsa -out private.pem 2048
# openssl rsa -in private.pem -pubout -out public.pem

class JWTManagerRS256:
    """Менеджер JWT с асимметричным шифрованием (RS256)."""
    
    def __init__(self, private_key_path: str, public_key_path: str):
        with open(private_key_path, 'rb') as f:
            self.private_key = f.read()
        
        with open(public_key_path, 'rb') as f:
            self.public_key = f.read()
    
    def create_token(self, data: Dict, expires_in: int = 3600) -> str:
        payload = data.copy()
        payload['exp'] = datetime.utcnow() + timedelta(seconds=expires_in)
        payload['iat'] = datetime.utcnow()
        
        return jwt.encode(payload, self.private_key, algorithm='RS256')
    
    def verify_token(self, token: str) -> Optional[Dict]:
        try:
            return jwt.decode(token, self.public_key, algorithms=['RS256'])
        except jwt.PyJWTError as e:
            print(f"Ошибка: {e}")
            return None
5. Аутентификация в FastAPI
5.1. Полный пример с FastAPI
python
# auth_api.py
from fastapi import FastAPI, HTTPException, Depends, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, EmailStr
from datetime import datetime, timedelta
from typing import Optional
import jwt

# ============================================================
# КОНФИГУРАЦИЯ
# ============================================================

SECRET_KEY = "your-super-secret-key-change-in-production"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# ============================================================
# МОДЕЛИ
# ============================================================

class UserLogin(BaseModel):
    email: EmailStr
    password: str

class UserRegister(BaseModel):
    name: str
    email: EmailStr
    password: str

class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    user_id: int
    email: str

class User(BaseModel):
    id: int
    name: str
    email: str
    is_active: bool = True

# ============================================================
# ХРАНИЛИЩЕ ПОЛЬЗОВАТЕЛЕЙ (имитация БД)
# ============================================================

users_db = {}
user_counter = 0


def get_user_by_email(email: str) -> Optional[dict]:
    """Поиск пользователя по email."""
    for user in users_db.values():
        if user['email'] == email:
            return user
    return None


def get_user_by_id(user_id: int) -> Optional[dict]:
    """Поиск пользователя по ID."""
    return users_db.get(user_id)


def create_user(name: str, email: str, password: str) -> dict:
    """Создание нового пользователя."""
    global user_counter
    user_counter += 1
    
    # В реальном приложении пароль нужно хэшировать!
    user = {
        'id': user_counter,
        'name': name,
        'email': email,
        'password': password,  # В реальности: hash(password)
        'is_active': True
    }
    users_db[user_counter] = user
    return user


# ============================================================
# JWT ФУНКЦИИ
# ============================================================

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    """Создание access-токена."""
    to_encode = data.copy()
    
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    
    to_encode.update({"exp": expire, "iat": datetime.utcnow()})
    
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)


def verify_access_token(token: str) -> Optional[TokenData]:
    """Верификация access-токена."""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id = payload.get("sub")
        email = payload.get("email")
        
        if user_id is None or email is None:
            return None
        
        return TokenData(user_id=int(user_id), email=email)
        
    except jwt.PyJWTError:
        return None


# ============================================================
# DEPENDENCIES
# ============================================================

security = HTTPBearer()


async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> User:
    """
    Получение текущего пользователя из токена.
    """
    token = credentials.credentials
    token_data = verify_access_token(token)
    
    if token_data is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный или истёкший токен",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    user = get_user_by_id(token_data.user_id)
    
    if user is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Пользователь не найден",
        )
    
    if not user.get('is_active', True):
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Пользователь заблокирован",
        )
    
    return User(
        id=user['id'],
        name=user['name'],
        email=user['email'],
        is_active=user['is_active']
    )


async def get_current_admin_user(current_user: User = Depends(get_current_user)) -> User:
    """Проверка прав администратора."""
    # В реальном приложении нужно проверять роль
    if current_user.email != "admin@example.com":
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Требуются права администратора"
        )
    return current_user


# ============================================================
# СОЗДАНИЕ ПРИЛОЖЕНИЯ
# ============================================================

app = FastAPI(title="Auth API", description="API с JWT аутентификацией")


# ============================================================
# ПУБЛИЧНЫЕ ЭНДПОИНТЫ (без аутентификации)
# ============================================================

@app.post("/register", response_model=User, status_code=201)
async def register(user_data: UserRegister):
    """Регистрация нового пользователя."""
    # Проверка существования пользователя
    if get_user_by_email(user_data.email):
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Пользователь с таким email уже существует"
        )
    
    # Создание пользователя
    user = create_user(
        name=user_data.name,
        email=user_data.email,
        password=user_data.password
    )
    
    return User(
        id=user['id'],
        name=user['name'],
        email=user['email']
    )


@app.post("/login", response_model=Token)
async def login(user_data: UserLogin):
    """Аутентификация пользователя."""
    # Поиск пользователя
    user = get_user_by_email(user_data.email)
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный email или пароль"
        )
    
    # Проверка пароля (в реальности используйте hashlib или bcrypt)
    if user['password'] != user_data.password:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный email или пароль"
        )
    
    # Создание токена
    access_token = create_access_token(
        data={"sub": user['id'], "email": user['email'], "role": "user"}
    )
    
    return Token(access_token=access_token, token_type="bearer")


# ============================================================
# ЗАЩИЩЁННЫЕ ЭНДПОИНТЫ (требуют токен)
# ============================================================

@app.get("/profile", response_model=User)
async def get_profile(current_user: User = Depends(get_current_user)):
    """Получение профиля текущего пользователя."""
    return current_user


@app.get("/users/me")
async def get_current_user_info(current_user: User = Depends(get_current_user)):
    """Получение информации о текущем пользователе."""
    return {
        "id": current_user.id,
        "name": current_user.name,
        "email": current_user.email,
        "is_active": current_user.is_active
    }


@app.get("/protected")
async def protected_route(current_user: User = Depends(get_current_user)):
    """Защищённый маршрут."""
    return {
        "message": f"Привет, {current_user.name}! Это защищённый маршрут.",
        "user_id": current_user.id
    }


@app.get("/admin")
async def admin_route(current_user: User = Depends(get_current_admin_user)):
    """Административный маршрут."""
    return {"message": f"Добро пожаловать в админ-панель, {current_user.name}!"}


# ============================================================
# ЗАПУСК
# ============================================================

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000)
5.2. Тестирование API
bash
# Регистрация
curl -X POST http://localhost:8000/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Иван","email":"ivan@example.com","password":"secret123"}'

# Логин
curl -X POST http://localhost:8000/login \
  -H "Content-Type: application/json" \
  -d '{"email":"ivan@example.com","password":"secret123"}'

# Ответ: {"access_token":"eyJhbG...","token_type":"bearer"}

# Доступ к защищённому маршруту
TOKEN="eyJhbGciOiJIUzI1NiIs..."
curl -X GET http://localhost:8000/protected \
  -H "Authorization: Bearer $TOKEN"
6. Refresh-токены
6.1. Зачем нужны Refresh-токены
text
┌─────────────────────────────────────────────────────────────────┐
│                     Access Token (короткий)                     │
│                 живёт 15-30 минут, для API запросов              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Refresh Token (долгий)                       │
│                живёт дни/недели, только для обновления           │
└─────────────────────────────────────────────────────────────────┘
6.2. Реализация Refresh-токенов
python
from fastapi import FastAPI, HTTPException, Depends, status
from fastapi.security import HTTPBearer
from pydantic import BaseModel
from datetime import datetime, timedelta
from typing import Optional, Dict
import jwt
import uuid

app = FastAPI()

SECRET_KEY = "your-secret-key"
REFRESH_SECRET_KEY = "your-refresh-secret-key"
ACCESS_TOKEN_EXPIRE_MINUTES = 15
REFRESH_TOKEN_EXPIRE_DAYS = 7

# Хранилище refresh-токенов (в реальности — Redis/БД)
refresh_tokens_db: Dict[str, dict] = {}


class TokenResponse(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str


class RefreshRequest(BaseModel):
    refresh_token: str


def create_access_token(data: dict) -> str:
    payload = data.copy()
    payload['exp'] = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    payload['type'] = 'access'
    return jwt.encode(payload, SECRET_KEY, algorithm='HS256')


def create_refresh_token(user_id: int) -> str:
    token_id = str(uuid.uuid4())
    payload = {
        'sub': user_id,
        'jti': token_id,
        'exp': datetime.utcnow() + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS),
        'type': 'refresh'
    }
    token = jwt.encode(payload, REFRESH_SECRET_KEY, algorithm='HS256')
    
    # Сохраняем токен в хранилище
    refresh_tokens_db[token_id] = {
        'user_id': user_id,
        'created_at': datetime.utcnow(),
        'expires_at': datetime.utcnow() + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS)
    }
    
    return token


def verify_refresh_token(token: str) -> Optional[int]:
    try:
        payload = jwt.decode(token, REFRESH_SECRET_KEY, algorithms=['HS256'])
        
        if payload.get('type') != 'refresh':
            return None
        
        token_id = payload.get('jti')
        user_id = payload.get('sub')
        
        # Проверяем, существует ли токен в хранилище
        if token_id not in refresh_tokens_db:
            return None
        
        if refresh_tokens_db[token_id]['user_id'] != user_id:
            return None
        
        return user_id
        
    except jwt.PyJWTError:
        return None


def revoke_refresh_token(token: str) -> bool:
    """Отзыв refresh-токена (при выходе из системы)."""
    try:
        payload = jwt.decode(token, REFRESH_SECRET_KEY, algorithms=['HS256'])
        token_id = payload.get('jti')
        if token_id in refresh_tokens_db:
            del refresh_tokens_db[token_id]
            return True
    except jwt.PyJWTError:
        pass
    return False


@app.post("/login", response_model=TokenResponse)
async def login(email: str, password: str):
    """Аутентификация, выдаёт access и refresh токены."""
    # Проверка логина (упрощённо)
    user_id = 1
    
    access_token = create_access_token({"sub": user_id})
    refresh_token = create_refresh_token(user_id)
    
    return TokenResponse(
        access_token=access_token,
        refresh_token=refresh_token,
        token_type="bearer"
    )


@app.post("/refresh", response_model=TokenResponse)
async def refresh_token(request: RefreshRequest):
    """Обновление access-токена по refresh-токену."""
    user_id = verify_refresh_token(request.refresh_token)
    
    if user_id is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный или истёкший refresh-токен"
        )
    
    # Создаём новые токены
    new_access_token = create_access_token({"sub": user_id})
    new_refresh_token = create_refresh_token(user_id)
    
    return TokenResponse(
        access_token=new_access_token,
        refresh_token=new_refresh_token,
        token_type="bearer"
    )


@app.post("/logout")
async def logout(request: RefreshRequest):
    """Выход из системы (отзыв refresh-токена)."""
    if revoke_refresh_token(request.refresh_token):
        return {"message": "Успешный выход"}
    return {"message": "Токен уже недействителен"}
7. Практические примеры
7.1. Хранение токенов на клиенте
javascript
// Сохранение токена после логина
async function login(email, password) {
    const response = await fetch('/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
    });
    
    const data = await response.json();
    
    // Сохраняем токен
    localStorage.setItem('access_token', data.access_token);
    localStorage.setItem('refresh_token', data.refresh_token);
    
    return data;
}

// Использование токена в запросах
async function getProfile() {
    const token = localStorage.getItem('access_token');
    
    const response = await fetch('/profile', {
        headers: {
            'Authorization': `Bearer ${token}`
        }
    });
    
    if (response.status === 401) {
        // Токен истёк, пробуем обновить
        await refreshToken();
        return getProfile(); // Повторяем запрос
    }
    
    return response.json();
}

// Обновление токена
async function refreshToken() {
    const refreshToken = localStorage.getItem('refresh_token');
    
    const response = await fetch('/refresh', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ refresh_token: refreshToken })
    });
    
    const data = await response.json();
    
    localStorage.setItem('access_token', data.access_token);
    localStorage.setItem('refresh_token', data.refresh_token);
    
    return data;
}

// Выход из системы
function logout() {
    localStorage.removeItem('access_token');
    localStorage.removeItem('refresh_token');
    window.location.href = '/login';
}
7.2. Middleware для проверки токена в FastAPI
python
from fastapi import FastAPI, Request, HTTPException
from starlette.middleware.base import BaseHTTPMiddleware

class AuthMiddleware(BaseHTTPMiddleware):
    """Middleware для проверки JWT-токена."""
    
    async def dispatch(self, request: Request, call_next):
        # Пропускаем публичные маршруты
        public_paths = ['/login', '/register', '/docs', '/openapi.json']
        
        if request.url.path in public_paths:
            return await call_next(request)
        
        # Получаем токен из заголовка
        auth_header = request.headers.get('Authorization')
        
        if not auth_header or not auth_header.startswith('Bearer '):
            return JSONResponse(
                status_code=401,
                content={"detail": "Требуется авторизация"}
            )
        
        token = auth_header.replace('Bearer ', '')
        
        # Верифицируем токен
        user = verify_access_token(token)
        
        if not user:
            return JSONResponse(
                status_code=401,
                content={"detail": "Неверный или истёкший токен"}
            )
        
        # Добавляем пользователя в request
        request.state.user = user
        
        return await call_next(request)


# Подключение middleware
app.add_middleware(AuthMiddleware)
8. Контрольные вопросы
В чём разница между аутентификацией и авторизацией?

Что такое JWT и из каких частей он состоит?

Какие claims являются зарезервированными в JWT?

Как проверить срок действия JWT-токена?

Зачем нужны Refresh-токены?

Как хранить JWT-токены на клиенте?

Чем отличается HS256 от RS256?

Как защитить API от подделки JWT?

Как реализовать выход из системы при использовании JWT?

Какие есть альтернативы JWT для аутентификации?

9. Практическое задание
Задание 1 (базовое)
Реализуйте JWT-аутентификацию для API заметок. Эндпоинты:

/register — регистрация

/login — вход

/notes — получить все заметки текущего пользователя

/notes — создать заметку

Задание 2 (среднее)
Добавьте Refresh-токены и обновление access-токенов. Реализуйте:

/refresh — обновление токена

/logout — выход из системы (отзыв refresh-токена)

Задание 3 (сложное)
Создайте полноценный проект с аутентификацией и ролями:

Роли: user, moderator, admin

Разные права доступа для разных ролей

Middleware для проверки токенов

Blacklist для отозванных токенов

10. Шпаргалка
python
# === УСТАНОВКА ===
pip install pyjwt

# === СОЗДАНИЕ ТОКЕНА ===
import jwt
from datetime import datetime, timedelta

payload = {"user_id": 1, "exp": datetime.utcnow() + timedelta(hours=1)}
token = jwt.encode(payload, "secret", algorithm="HS256")

# === ВЕРИФИКАЦИЯ ===
try:
    payload = jwt.decode(token, "secret", algorithms=["HS256"])
except jwt.ExpiredSignatureError:
    print("Токен истёк")
except jwt.InvalidTokenError:
    print("Неверный токен")

# === FASTAPI DEPENDENCY ===
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer

security = HTTPBearer()

async def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    token = credentials.credentials
    # верификация токена
    return user
Итог лекции
Вы сегодня:

✅ Изучили основы аутентификации и авторизации

✅ Освоили работу с JWT-токенами (создание, верификация)

✅ Реализовали аутентификацию в FastAPI

✅ Изучили Refresh-токены и их применение

✅ Разобрали практические примеры

Теперь вы можете защитить свои API с помощью JWT-аутентификации!

