# ПЗ 2.49. Добавление аутентификации и авторизации в проект

**Тема:** Аутентификация, авторизация, JWT, роли, защита API

**Цель работы:**  
Научиться добавлять аутентификацию и авторизацию в существующий веб-проект, реализовать регистрацию и вход пользователей, управление ролями и защиту эндпоинтов.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `fastapi`, `uvicorn`, `sqlalchemy`, `pydantic`, `python-jose`, `passlib`, `python-dotenv`

```bash
pip install fastapi uvicorn sqlalchemy pydantic python-jose[cryptography] passlib[bcrypt] python-dotenv
Главная мысль: Аутентификация отвечает на вопрос "кто ты?", авторизация — "что тебе можно?". Без них любой API — открытая книга для всех.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Аутентификация vs Авторизация
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    АУТЕНТИФИКАЦИЯ vs АВТОРИЗАЦИЯ                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    АУТЕНТИФИКАЦИЯ                                   │   │
│  │                                                                     │   │
│  │   "Кто ты?"                                                         │   │
│  │                                                                     │   │
│  │   • Регистрация (создание учётной записи)                          │   │
│  │   • Логин (вход с паролем)                                          │   │
│  │   • JWT токен (подтверждение личности)                              │   │
│  │   • Проверка токена в каждом запросе                                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    АВТОРИЗАЦИЯ                                      │   │
│  │                                                                     │   │
│  │   "Что тебе можно?"                                                 │   │
│  │                                                                     │   │
│  │   • Проверка прав доступа                                           │   │
│  │   • Роли пользователей (admin, user, moderator)                     │   │
│  │   • Разрешения (просмотр, создание, редактирование, удаление)       │   │
│  │   • Владелец ресурса (только свои данные)                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
1.2. Компоненты системы аутентификации
Компонент	Ответственность	Технологии
Модель User	Хранение данных пользователя	SQLAlchemy
Хэширование	Безопасное хранение паролей	passlib/bcrypt
JWT	Создание и проверка токенов	python-jose
Зависимости	Проверка токена в эндпоинтах	FastAPI Depends
Middleware	Глобальная проверка	Starlette
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая добавление аутентификации и авторизации в проект.

Техническое задание (нулевой вариант)
Добавьте аутентификацию и авторизацию в существующий проект Task Manager. Реализуйте:

Регистрацию пользователей (POST /auth/register)

Вход в систему (POST /auth/login) с выдачей JWT

Защиту эндпоинтов задач (только для авторизованных пользователей)

Права доступа: пользователь может управлять только своими задачами

Роли: admin может видеть все задачи, user — только свои

Дополнение к существующему проекту
Файл app/models/user.py (обновление)
python
from sqlalchemy import Column, String, Boolean, Integer, DateTime
from sqlalchemy.orm import relationship
from datetime import datetime
from passlib.context import CryptContext
import re

from ..database import Base

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


class UserRole:
    """Роли пользователей."""
    USER = "user"
    ADMIN = "admin"
    MODERATOR = "moderator"


class User(Base):
    """Модель пользователя с аутентификацией."""
    
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(50), unique=True, nullable=False, index=True)
    email = Column(String(100), unique=True, nullable=False, index=True)
    password_hash = Column(String(255), nullable=False)
    role = Column(String(20), default=UserRole.USER)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
    
    # Связь с задачами
    tasks = relationship("Task", back_populates="user", cascade="all, delete-orphan")
    
    def set_password(self, password: str):
        """Установка хэша пароля."""
        self.password_hash = pwd_context.hash(password)
    
    def verify_password(self, password: str) -> bool:
        """Проверка пароля."""
        return pwd_context.verify(password, self.password_hash)
    
    def is_admin(self) -> bool:
        """Является ли пользователь администратором."""
        return self.role == UserRole.ADMIN
    
    def can_manage_task(self, task_user_id: int) -> bool:
        """Может ли пользователь управлять задачей."""
        return self.is_admin() or self.id == task_user_id
    
    @classmethod
    def validate_username(cls, username: str) -> bool:
        """Валидация имени пользователя."""
        return bool(re.match(r'^[a-zA-Z0-9_]{3,50}$', username))
    
    @classmethod
    def validate_password(cls, password: str) -> bool:
        """Валидация пароля."""
        return len(password) >= 6 and re.search(r'[A-Za-z]', password) and re.search(r'[0-9]', password)
    
    def __repr__(self):
        return f"<User(id={self.id}, username={self.username}, role={self.role})>"
Файл app/schemas/auth.py
python
from pydantic import BaseModel, Field, EmailStr, validator
from typing import Optional
from datetime import datetime


class UserRegister(BaseModel):
    """Схема регистрации пользователя."""
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    password: str = Field(..., min_length=6)
    confirm_password: str = Field(..., min_length=6)
    
    @validator('username')
    def username_valid(cls, v):
        import re
        if not re.match(r'^[a-zA-Z0-9_]+$', v):
            raise ValueError('Username can only contain letters, numbers and underscore')
        return v.lower()
    
    @validator('password')
    def password_valid(cls, v):
        if len(v) < 6:
            raise ValueError('Password must be at least 6 characters')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain at least one digit')
        if not any(c.isalpha() for c in v):
            raise ValueError('Password must contain at least one letter')
        return v
    
    @validator('confirm_password')
    def passwords_match(cls, v, values):
        if 'password' in values and v != values['password']:
            raise ValueError('Passwords do not match')
        return v


class UserLogin(BaseModel):
    """Схема входа в систему."""
    username: str
    password: str


class TokenResponse(BaseModel):
    """Схема ответа с токеном."""
    access_token: str
    token_type: str = "bearer"
    expires_in: int
    user_id: int
    username: str
    role: str


class UserResponse(BaseModel):
    """Схема ответа с данными пользователя."""
    id: int
    username: str
    email: str
    role: str
    is_active: bool
    created_at: datetime
    
    class Config:
        from_attributes = True


class PasswordChange(BaseModel):
    """Схема смены пароля."""
    old_password: str
    new_password: str = Field(..., min_length=6)
    confirm_password: str = Field(..., min_length=6)
    
    @validator('new_password')
    def password_valid(cls, v):
        if len(v) < 6:
            raise ValueError('Password must be at least 6 characters')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain at least one digit')
        return v
    
    @validator('confirm_password')
    def passwords_match(cls, v, values):
        if 'new_password' in values and v != values['new_password']:
            raise ValueError('Passwords do not match')
        return v


class UserUpdate(BaseModel):
    """Схема обновления пользователя."""
    username: Optional[str] = Field(None, min_length=3, max_length=50)
    email: Optional[EmailStr] = None
    role: Optional[str] = None
    
    @validator('username')
    def username_valid(cls, v):
        if v:
            import re
            if not re.match(r'^[a-zA-Z0-9_]+$', v):
                raise ValueError('Username can only contain letters, numbers and underscore')
        return v
Файл app/services/auth_service.py
python
from sqlalchemy.orm import Session
from datetime import datetime, timedelta
from typing import Optional, Dict, Any
from jose import JWTError, jwt

from ..config import settings
from ..models.user import User, UserRole
from ..repositories.user_repository import UserRepository
from ..schemas.auth import UserRegister, UserLogin, PasswordChange
from ..utils.exceptions import BusinessError, UnauthorizedError


class AuthService:
    """Сервис аутентификации и авторизации."""
    
    def __init__(self, db: Session):
        self.db = db
        self.repo = UserRepository(db)
    
    def register(self, user_data: UserRegister) -> User:
        """Регистрация нового пользователя."""
        # Проверка существования пользователя
        if self.repo.get_by_username(user_data.username):
            raise BusinessError("Username already exists")
        
        if self.repo.get_by_email(user_data.email):
            raise BusinessError("Email already exists")
        
        # Создание пользователя
        user = self.repo.create(
            username=user_data.username,
            email=user_data.email,
            role=UserRole.USER,
            is_active=True
        )
        user.set_password(user_data.password)
        self.db.commit()
        
        return user
    
    def login(self, login_data: UserLogin) -> Dict[str, Any]:
        """Аутентификация пользователя."""
        # Поиск пользователя
        user = self.repo.get_by_username(login_data.username)
        if not user:
            raise UnauthorizedError("Invalid username or password")
        
        # Проверка пароля
        if not user.verify_password(login_data.password):
            raise UnauthorizedError("Invalid username or password")
        
        # Проверка активности
        if not user.is_active:
            raise UnauthorizedError("Account is disabled. Please contact administrator")
        
        # Создание токена
        access_token = self._create_access_token(
            data={
                "sub": str(user.id),
                "username": user.username,
                "role": user.role
            }
        )
        
        return {
            "access_token": access_token,
            "token_type": "bearer",
            "expires_in": settings.ACCESS_TOKEN_EXPIRE_MINUTES * 60,
            "user_id": user.id,
            "username": user.username,
            "role": user.role
        }
    
    def _create_access_token(self, data: dict) -> str:
        """Создание JWT токена."""
        to_encode = data.copy()
        expire = datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
        to_encode.update({"exp": expire})
        return jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)
    
    def change_password(self, user_id: int, password_data: PasswordChange) -> bool:
        """Смена пароля."""
        user = self.repo.get_by_id(user_id)
        if not user:
            raise BusinessError("User not found")
        
        if not user.verify_password(password_data.old_password):
            raise BusinessError("Invalid old password")
        
        user.set_password(password_data.new_password)
        self.db.commit()
        
        return True
    
    def get_current_user(self, token: str) -> User:
        """Получение текущего пользователя из токена."""
        try:
            payload = jwt.decode(
                token, 
                settings.SECRET_KEY, 
                algorithms=[settings.ALGORITHM]
            )
            user_id = int(payload.get("sub"))
            user = self.repo.get_by_id(user_id)
            
            if not user or not user.is_active:
                raise UnauthorizedError("User not found or inactive")
            
            return user
            
        except JWTError:
            raise UnauthorizedError("Invalid token")
        except Exception:
            raise UnauthorizedError("Authentication failed")
Файл app/api/auth.py
python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from typing import Dict, Any

from ..database import get_db
from ..services.auth_service import AuthService
from ..schemas.auth import (
    UserRegister, UserLogin, TokenResponse, 
    UserResponse, PasswordChange, UserUpdate
)
from ..utils.auth import get_current_user
from ..models.user import User

router = APIRouter(prefix="/auth", tags=["Authentication"])


@router.post("/register", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
def register(
    user_data: UserRegister,
    db: Session = Depends(get_db)
):
    """
    Регистрация нового пользователя.
    
    - **username**: уникальное имя пользователя (только буквы, цифры, _)
    - **email**: уникальный email
    - **password**: пароль (мин. 6 символов, буквы и цифры)
    - **confirm_password**: подтверждение пароля
    """
    service = AuthService(db)
    return service.register(user_data)


@router.post("/login", response_model=TokenResponse)
def login(
    login_data: UserLogin,
    db: Session = Depends(get_db)
):
    """
    Вход в систему.
    
    - **username**: имя пользователя
    - **password**: пароль
    
    Возвращает JWT токен для доступа к API.
    """
    service = AuthService(db)
    return service.login(login_data)


@router.post("/change-password")
def change_password(
    password_data: PasswordChange,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """
    Смена пароля текущего пользователя.
    
    Требует авторизации (токен в заголовке Authorization: Bearer <token>)
    """
    service = AuthService(db)
    service.change_password(current_user.id, password_data)
    return {"message": "Password changed successfully"}


@router.get("/me", response_model=UserResponse)
def get_current_user_info(
    current_user: User = Depends(get_current_user)
):
    """Получение информации о текущем пользователе."""
    return current_user


@router.put("/me", response_model=UserResponse)
def update_current_user(
    user_data: UserUpdate,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Обновление данных текущего пользователя."""
    service = AuthService(db)
    return service.update_user(current_user.id, user_data)


@router.post("/logout")
def logout(current_user: User = Depends(get_current_user)):
    """
    Выход из системы.
    
    Токен становится недействительным (клиент должен удалить его).
    Для полной защиты рекомендуется реализовать черный список токенов.
    """
    return {"message": "Successfully logged out"}
Файл app/utils/auth.py
python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.orm import Session
from typing import Optional

from ..database import get_db
from ..services.auth_service import AuthService
from ..models.user import User, UserRole

security = HTTPBearer(auto_error=False)


async def get_current_user(
    credentials: Optional[HTTPAuthorizationCredentials] = Depends(security),
    db: Session = Depends(get_db)
) -> User:
    """
    Dependency для получения текущего пользователя.
    """
    if not credentials:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Not authenticated",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    service = AuthService(db)
    return service.get_current_user(credentials.credentials)


async def get_current_active_user(
    current_user: User = Depends(get_current_user)
) -> User:
    """
    Dependency для получения активного пользователя.
    """
    if not current_user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="User account is disabled"
        )
    return current_user


async def get_current_admin_user(
    current_user: User = Depends(get_current_active_user)
) -> User:
    """
    Dependency для получения администратора.
    """
    if current_user.role != UserRole.ADMIN:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Admin access required"
        )
    return current_user


async def get_current_moderator_user(
    current_user: User = Depends(get_current_active_user)
) -> User:
    """
    Dependency для получения модератора или администратора.
    """
    if current_user.role not in [UserRole.ADMIN, UserRole.MODERATOR]:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Moderator access required"
        )
    return current_user


def check_ownership(resource_user_id: int, current_user: User) -> bool:
    """
    Проверка владельца ресурса.
    """
    return current_user.is_admin() or current_user.id == resource_user_id
Файл app/api/tasks.py (обновление с аутентификацией)
python
from fastapi import APIRouter, Depends, Query, HTTPException, status
from sqlalchemy.orm import Session
from typing import Optional, List

from ..database import get_db
from ..services.task_service import TaskService
from ..schemas.task import TaskCreate, TaskUpdate, TaskResponse, TaskListResponse
from ..utils.auth import get_current_active_user, get_current_admin_user, check_ownership
from ..models.user import User, UserRole
from ..models.task import TaskStatus, TaskPriority

router = APIRouter(prefix="/tasks", tags=["Tasks"])


@router.post("/", response_model=TaskResponse, status_code=status.HTTP_201_CREATED)
def create_task(
    task_data: TaskCreate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_active_user)
):
    """
    Создание новой задачи.
    
    Требует авторизации. Задача автоматически привязывается к текущему пользователю.
    """
    service = TaskService(db)
    return service.create_task(current_user.id, task_data)


@router.get("/", response_model=TaskListResponse)
def get_tasks(
    page: int = Query(1, ge=1),
    limit: int = Query(20, ge=1, le=100),
    status: Optional[TaskStatus] = None,
    priority: Optional[TaskPriority] = None,
    user_id: Optional[int] = None,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_active_user)
):
    """
    Получение списка задач.
    
    - Обычный пользователь видит только свои задачи
    - Администратор видит задачи всех пользователей
    - Поддержка пагинации, фильтрации по статусу и приоритету
    """
    service = TaskService(db)
    
    # Определяем, чьи задачи показывать
    target_user_id = current_user.id
    if current_user.is_admin() and user_id:
        target_user_id = user_id
    
    skip = (page - 1) * limit
    return service.get_user_tasks(
        user_id=target_user_id,
        skip=skip,
        limit=limit,
        status=status,
        priority=priority
    )


@router.get("/{task_id}", response_model=TaskResponse)
def get_task(
    task_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_active_user)
):
    """
    Получение задачи по ID.
    
    Доступна только владельцу задачи или администратору.
    """
    service = TaskService(db)
    task = service.get_task(task_id, current_user.id)
    
    # Проверка прав доступа (дополнительная защита)
    if not check_ownership(task.user_id, current_user):
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Access denied"
        )
    
    return task


@router.put("/{task_id}", response_model=TaskResponse)
def update_task(
    task_id: int,
    task_data: TaskUpdate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_active_user)
):
    """
    Обновление задачи.
    
    Доступно только владельцу задачи или администратору.
    """
    service = TaskService(db)
    return service.update_task(task_id, current_user.id, task_data)


@router.delete("/{task_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_task(
    task_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_active_user)
):
    """
    Удаление задачи.
    
    Доступно только владельцу задачи или администратору.
    """
    service = TaskService(db)
    service.delete_task(task_id, current_user.id)


@router.post("/{task_id}/complete", response_model=TaskResponse)
def complete_task(
    task_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_active_user)
):
    """
    Отметка задачи как выполненной.
    
    Доступно только владельцу задачи или администратору.
    """
    service = TaskService(db)
    return service.complete_task(task_id, current_user.id)


# Административные эндпоинты (только для админов)
@router.get("/admin/all", response_model=List[TaskResponse])
def get_all_tasks_admin(
    skip: int = 0,
    limit: int = 100,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_admin_user)
):
    """
    Получение всех задач (только для администраторов).
    """
    service = TaskService(db)
    return service.get_all_tasks(skip, limit)


@router.get("/admin/users/{user_id}/tasks", response_model=TaskListResponse)
def get_user_tasks_admin(
    user_id: int,
    page: int = Query(1, ge=1),
    limit: int = Query(20, ge=1, le=100),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_admin_user)
):
    """
    Получение задач конкретного пользователя (только для администраторов).
    """
    service = TaskService(db)
    skip = (page - 1) * limit
    return service.get_user_tasks(user_id=user_id, skip=skip, limit=limit)


@router.get("/stats/my")
def get_my_stats(
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_active_user)
):
    """
    Статистика по задачам текущего пользователя.
    """
    service = TaskService(db)
    return service.get_statistics(current_user.id)
Файл app/api/admin.py (новый модуль для административных функций)
python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from typing import List

from ..database import get_db
from ..services.user_service import UserService
from ..services.auth_service import AuthService
from ..schemas.auth import UserResponse, UserUpdate
from ..utils.auth import get_current_admin_user
from ..models.user import User

router = APIRouter(prefix="/admin", tags=["Admin"])


@router.get("/users", response_model=List[UserResponse])
def get_all_users(
    skip: int = 0,
    limit: int = 100,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_admin_user)
):
    """Получение списка всех пользователей (только для администраторов)."""
    service = UserService(db)
    return service.get_all_users(skip, limit)


@router.get("/users/{user_id}", response_model=UserResponse)
def get_user_by_id(
    user_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_admin_user)
):
    """Получение пользователя по ID (только для администраторов)."""
    service = UserService(db)
    return service.get_user(user_id)


@router.put("/users/{user_id}", response_model=UserResponse)
def update_user_by_admin(
    user_id: int,
    user_data: UserUpdate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_admin_user)
):
    """Обновление пользователя (только для администраторов)."""
    service = UserService(db)
    return service.update_user_by_admin(user_id, user_data)


@router.delete("/users/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_user_by_admin(
    user_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_admin_user)
):
    """Удаление пользователя (только для администраторов)."""
    service = UserService(db)
    service.delete_user(user_id)


@router.post("/users/{user_id}/activate")
def activate_user(
    user_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_admin_user)
):
    """Активация пользователя (только для администраторов)."""
    service = UserService(db)
    return service.activate_user(user_id)


@router.post("/users/{user_id}/deactivate")
def deactivate_user(
    user_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_admin_user)
):
    """Деактивация пользователя (только для администраторов)."""
    service = UserService(db)
    return service.deactivate_user(user_id)
Файл app/main.py (обновление с регистрацией роутеров)
python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager

from .database import engine, Base
from .config import settings
from .api import tasks, auth, admin

# Создание таблиц
Base.metadata.create_all(bind=engine)


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Управление жизненным циклом приложения."""
    print("🚀 Starting Task Manager API with Authentication")
    yield
    print("🛑 Shutting down...")


app = FastAPI(
    title="Task Manager API",
    description="API для управления задачами с аутентификацией и авторизацией",
    version="2.0.0",
    lifespan=lifespan
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
    allow_credentials=True,
)

# Регистрация роутеров
app.include_router(auth.router)
app.include_router(tasks.router)
app.include_router(admin.router)


@app.get("/")
def root():
    return {
        "message": "Task Manager API",
        "version": "2.0.0",
        "auth": {
            "register": "POST /auth/register",
            "login": "POST /auth/login",
            "docs": "/docs"
        }
    }


@app.get("/health")
def health_check():
    return {"status": "healthy"}
Тестирование API
bash
# 1. Регистрация
curl -X POST http://localhost:8000/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "ivan",
    "email": "ivan@example.com",
    "password": "secret123",
    "confirm_password": "secret123"
  }'

# 2. Логин
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "ivan", "password": "secret123"}'

# Ответ:
# {
#   "access_token": "eyJhbGciOiJIUzI1NiIs...",
#   "token_type": "bearer",
#   "expires_in": 1800,
#   "user_id": 1,
#   "username": "ivan",
#   "role": "user"
# }

# 3. Создание задачи (с токеном)
TOKEN="your-token-here"
curl -X POST http://localhost:8000/tasks/ \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title": "Изучить FastAPI", "priority": "high"}'

# 4. Получение задач
curl -X GET "http://localhost:8000/tasks/?page=1&limit=10" \
  -H "Authorization: Bearer $TOKEN"

# 5. Смена пароля
curl -X POST http://localhost:8000/auth/change-password \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "old_password": "secret123",
    "new_password": "newsecret456",
    "confirm_password": "newsecret456"
  }'

# 6. Получение информации о текущем пользователе
curl -X GET http://localhost:8000/auth/me \
  -H "Authorization: Bearer $TOKEN"
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (только аутентификация)

Варианты 9-17: средний (аутентификация + роли)

Варианты 18-25: сложный (полная система с административной панелью)

Варианты 1-8 (Базовый уровень)
№	Ресурс	Аутентификация	Авторизация
1	Заметки	JWT	Владелец
2	Контакты	JWT	Владелец
3	Задачи	JWT	Владелец
4	Книги	JWT	Владелец
5	Фильмы	JWT	Владелец
6	События	JWT	Владелец
7	Расходы	JWT	Владелец
8	Дневник	JWT	Владелец
Варианты 9-17 (Средний уровень)
№	Ресурс	Роли	Дополнительно
9	Посты	user, admin	Модерация
10	Товары	user, manager	Управление запасами
11	Заказы	user, admin	Статусы
12	Клиенты	manager, admin	CRM
13	Сотрудники	HR, admin	Кадры
14	Курсы	student, teacher	Образование
15	Проекты	member, lead	Управление
16	Тикеты	user, support	Поддержка
17	Комментарии	user, moderator	Модерация
Варианты 18-25 (Сложный уровень)
№	Тема	Роли	Особенности
18	Блог	reader, author, admin	Публикация, модерация
19	Магазин	customer, manager, admin	Корзина, заказы
20	Форум	user, moderator, admin	Темы, сообщения
21	Библиотека	reader, librarian, admin	Выдача книг
22	CRM	sales, manager, admin	Сделки, отчёты
23	Образование	student, teacher, admin	Оценки, курсы
24	Доставка	client, courier, admin	Заказы, трекинг
25	Соцсеть	user, moderator, admin	Посты, друзья
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Аутентификация не работает
3 (удовлетворительно)	Регистрация и логин работают
4 (хорошо)	+ Защита эндпоинтов, владелец ресурса
5 (отлично)	+ Роли, админ-панель, документация
5. Шпаргалка
python
# === ЗАВИСИМОСТИ ДЛЯ АУТЕНТИФИКАЦИИ ===
from fastapi import Depends
from .utils.auth import get_current_user, get_current_admin_user

# Защищённый эндпоинт
@app.get("/protected")
def protected(current_user: User = Depends(get_current_user)):
    return {"user": current_user.username}

# Только для админов
@app.get("/admin")
def admin_only(current_user: User = Depends(get_current_admin_user)):
    return {"message": "Admin access granted"}

# === ПРОВЕРКА ВЛАДЕЛЬЦА ===
if resource.user_id != current_user.id and not current_user.is_admin():
    raise HTTPException(status_code=403, detail="Access denied")

# === JWT ===
from jose import jwt
token = jwt.encode({"sub": user_id}, SECRET_KEY, algorithm="HS256")
payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
Карточка студента
text
ПЗ 2.49. ДОБАВЛЕНИЕ АУТЕНТИФИКАЦИИ И АВТОРИЗАЦИИ В ПРОЕКТ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== РЕСУРСЫ ===

1. _____________
2. _____________
3. _____________

=== РОЛИ ===

□ user (обычный пользователь)
□ moderator (модератор)
□ admin (администратор)

=== АУТЕНТИФИКАЦИЯ ===

□ Регистрация (/auth/register)
□ Логин (/auth/login)
□ JWT токены
□ Защита эндпоинтов

=== АВТОРИЗАЦИЯ ===

□ Владелец ресурса
□ Ролевой доступ
□ Административные эндпоинты

=== ОТЧЁТ ===

Количество эндпоинтов: _____
Из них защищённых: _____
Ролей: _____

Дата выполнения: _____________
