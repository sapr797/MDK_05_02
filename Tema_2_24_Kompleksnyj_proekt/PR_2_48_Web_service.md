# ПЗ 2.48. Реализация полноценного веб-сервиса с API, БД, валидацией

**Тема:** Комплексный проект, интеграция API, базы данных, валидации

**Цель работы:**  
Разработать полноценный веб-сервис с использованием FastAPI, SQLAlchemy, Pydantic, включающий CRUD-операции, валидацию данных, обработку ошибок и документацию.

**Время выполнения:** 120 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `fastapi`, `uvicorn`, `sqlalchemy`, `pydantic`, `python-dotenv`

```bash
pip install fastapi uvicorn sqlalchemy pydantic python-dotenv
Главная мысль: Профессиональный веб-сервис — это не просто код, а продуманная архитектура с разделением ответственности, валидацией и обработкой ошибок.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Архитектура веб-сервиса
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    АРХИТЕКТУРА ВЕБ-СЕРВИСА                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         API СЛОЙ (FastAPI)                          │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │   │
│  │  │  Routers    │  │  Schemas    │  │  Dependencies│                 │   │
│  │  │  (эндпоинты)│  │  (Pydantic) │  │  (Depends)   │                 │   │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                 │   │
│  │         │                │                │                         │   │
│  │         └────────────────┼────────────────┘                         │   │
│  │                          │                                          │   │
│  │                          ▼                                          │   │
│  │  ┌─────────────────────────────────────────────────────────────┐   │   │
│  │  │                    СЕРВИСНЫЙ СЛОЙ (Services)                 │   │   │
│  │  │              Бизнес-логика, валидация, обработка             │   │   │
│  │  └───────────────────────────────┬─────────────────────────────┘   │   │
│  └──────────────────────────────────┼─────────────────────────────────┘   │
│                                     │                                      │
│  ┌──────────────────────────────────┼─────────────────────────────────┐   │
│  │                                 │                                  │   │
│  │  ┌──────────────────────────────┼──────────────────────────────┐   │   │
│  │  │                              ▼                              │   │   │
│  │  │  ┌─────────────────────────────────────────────────────┐    │   │   │
│  │  │  │              СЛОЙ ДОСТУПА К ДАННЫМ (DAO/Repository)  │    │   │   │
│  │  │  │                   SQLAlchemy ORM                    │    │   │   │
│  │  │  └─────────────────────────────────────────────────────┘    │   │   │
│  │  │                              │                              │   │   │
│  │  │                              ▼                              │   │   │
│  │  │  ┌─────────────────────────────────────────────────────┐    │   │   │
│  │  │  │                   БАЗА ДАННЫХ                       │    │   │   │
│  │  │  │                   (SQLite/PostgreSQL)               │    │   │   │
│  │  │  └─────────────────────────────────────────────────────┘    │   │   │
│  │  └──────────────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
1.2. Компоненты веб-сервиса
Компонент	Ответственность	Технологии
Модели (Models)	Описание структуры БД	SQLAlchemy
Схемы (Schemas)	Валидация входящих/исходящих данных	Pydantic
Репозиторий (Repository)	CRUD операции с БД	SQLAlchemy
Сервис (Service)	Бизнес-логика	Python
Роутеры (Routers)	Эндпоинты API	FastAPI
Зависимости (Dependencies)	Общие компоненты	FastAPI Depends
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая реализацию полноценного веб-сервиса.

Техническое задание (нулевой вариант)
Разработайте веб-сервис для управления задачами (Task Manager) с поддержкой:

CRUD операций для задач

Пользователей с аутентификацией

Категорий задач

Валидации всех входных данных

Обработки ошибок

Автоматической документации

Реализация
Структура проекта
text
task_manager/
├── app/
│   ├── __init__.py
│   ├── main.py                 # Точка входа
│   ├── config.py               # Конфигурация
│   ├── database.py             # Подключение к БД
│   │
│   ├── models/                 # SQLAlchemy модели
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── task.py
│   │   └── category.py
│   │
│   ├── schemas/                # Pydantic схемы
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── task.py
│   │   └── category.py
│   │
│   ├── repositories/           # CRUD операции
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── user_repository.py
│   │   ├── task_repository.py
│   │   └── category_repository.py
│   │
│   ├── services/               # Бизнес-логика
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   ├── task_service.py
│   │   └── category_service.py
│   │
│   ├── api/                    # API роутеры
│   │   ├── __init__.py
│   │   ├── users.py
│   │   ├── tasks.py
│   │   └── categories.py
│   │
│   └── utils/                  # Утилиты
│       ├── __init__.py
│       ├── auth.py
│       └── exceptions.py
│
├── requirements.txt
├── .env
└── .gitignore
Файл app/config.py
python
import os
from dotenv import load_dotenv

load_dotenv()


class Settings:
    """Настройки приложения."""
    
    # Приложение
    APP_NAME = os.getenv("APP_NAME", "Task Manager API")
    APP_VERSION = os.getenv("APP_VERSION", "1.0.0")
    DEBUG = os.getenv("DEBUG", "False").lower() == "true"
    
    # База данных
    DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./task_manager.db")
    
    # Безопасность
    SECRET_KEY = os.getenv("SECRET_KEY", "your-secret-key-here")
    ALGORITHM = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES = int(os.getenv("ACCESS_TOKEN_EXPIRE_MINUTES", "30"))
    
    # Пагинация
    DEFAULT_PAGE_SIZE = int(os.getenv("DEFAULT_PAGE_SIZE", "20"))


settings = Settings()
Файл app/database.py
python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from typing import Generator

from .config import settings

# Создание движка
engine = create_engine(
    settings.DATABASE_URL,
    connect_args={"check_same_thread": False} if "sqlite" in settings.DATABASE_URL else {}
)

# Фабрика сессий
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Базовый класс для моделей
Base = declarative_base()


def get_db() -> Generator[Session, None, None]:
    """Dependency для получения сессии БД."""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
Файл app/models/base.py
python
from sqlalchemy import Column, Integer, DateTime
from datetime import datetime
from ..database import Base


class BaseModel(Base):
    """Абстрактная базовая модель с общими полями."""
    
    __abstract__ = True
    
    id = Column(Integer, primary_key=True, index=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
Файл app/models/user.py
python
from sqlalchemy import Column, String, Boolean
from sqlalchemy.orm import relationship
from passlib.context import CryptContext

from .base import BaseModel
from ..database import Base

# Настройка хэширования паролей
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


class User(BaseModel):
    """Модель пользователя."""
    
    __tablename__ = "users"
    
    username = Column(String(50), unique=True, nullable=False, index=True)
    email = Column(String(100), unique=True, nullable=False, index=True)
    password_hash = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    is_admin = Column(Boolean, default=False)
    
    # Связи
    tasks = relationship("Task", back_populates="user", cascade="all, delete-orphan")
    
    def set_password(self, password: str):
        """Установка хэша пароля."""
        self.password_hash = pwd_context.hash(password)
    
    def verify_password(self, password: str) -> bool:
        """Проверка пароля."""
        return pwd_context.verify(password, self.password_hash)
Файл app/models/category.py
python
from sqlalchemy import Column, String
from sqlalchemy.orm import relationship

from .base import BaseModel


class Category(BaseModel):
    """Модель категории."""
    
    __tablename__ = "categories"
    
    name = Column(String(50), unique=True, nullable=False, index=True)
    description = Column(String(200))
    
    # Связи
    tasks = relationship("Task", back_populates="category")
Файл app/models/task.py
python
from sqlalchemy import Column, String, Text, Integer, ForeignKey, Boolean
from sqlalchemy.orm import relationship
from enum import Enum

from .base import BaseModel


class TaskPriority(str, Enum):
    """Приоритет задачи."""
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"


class TaskStatus(str, Enum):
    """Статус задачи."""
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    COMPLETED = "completed"


class Task(BaseModel):
    """Модель задачи."""
    
    __tablename__ = "tasks"
    
    title = Column(String(200), nullable=False, index=True)
    description = Column(Text)
    priority = Column(String(20), default=TaskPriority.MEDIUM)
    status = Column(String(20), default=TaskStatus.PENDING)
    
    # Внешние ключи
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    category_id = Column(Integer, ForeignKey("categories.id"), nullable=True)
    
    # Связи
    user = relationship("User", back_populates="tasks")
    category = relationship("Category", back_populates="tasks")
Файл app/schemas/user.py
python
from pydantic import BaseModel, Field, EmailStr, validator
from datetime import datetime
from typing import Optional


class UserCreate(BaseModel):
    """Схема для создания пользователя."""
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    password: str = Field(..., min_length=6)
    
    @validator('username')
    def username_alphanumeric(cls, v):
        if not v.isalnum():
            raise ValueError('Username must contain only letters and numbers')
        return v.lower()


class UserUpdate(BaseModel):
    """Схема для обновления пользователя."""
    username: Optional[str] = Field(None, min_length=3, max_length=50)
    email: Optional[EmailStr] = None
    password: Optional[str] = Field(None, min_length=6)
    is_active: Optional[bool] = None


class UserResponse(BaseModel):
    """Схема ответа с пользователем."""
    id: int
    username: str
    email: str
    is_active: bool
    is_admin: bool
    created_at: datetime
    
    class Config:
        orm_mode = True


class UserLogin(BaseModel):
    """Схема для входа."""
    username: str
    password: str


class TokenResponse(BaseModel):
    """Схема ответа с токеном."""
    access_token: str
    token_type: str = "bearer"
Файл app/schemas/category.py
python
from pydantic import BaseModel, Field
from datetime import datetime
from typing import Optional


class CategoryCreate(BaseModel):
    """Схема для создания категории."""
    name: str = Field(..., min_length=1, max_length=50)
    description: Optional[str] = Field(None, max_length=200)


class CategoryUpdate(BaseModel):
    """Схема для обновления категории."""
    name: Optional[str] = Field(None, min_length=1, max_length=50)
    description: Optional[str] = Field(None, max_length=200)


class CategoryResponse(BaseModel):
    """Схема ответа с категорией."""
    id: int
    name: str
    description: Optional[str]
    created_at: datetime
    
    class Config:
        orm_mode = True
Файл app/schemas/task.py
python
from pydantic import BaseModel, Field, validator
from datetime import datetime
from typing import Optional
from ..models.task import TaskPriority, TaskStatus


class TaskCreate(BaseModel):
    """Схема для создания задачи."""
    title: str = Field(..., min_length=1, max_length=200)
    description: Optional[str] = None
    priority: TaskPriority = TaskPriority.MEDIUM
    category_id: Optional[int] = None
    
    @validator('category_id')
    def validate_category(cls, v):
        if v is not None and v <= 0:
            raise ValueError('category_id must be positive')
        return v


class TaskUpdate(BaseModel):
    """Схема для обновления задачи."""
    title: Optional[str] = Field(None, min_length=1, max_length=200)
    description: Optional[str] = None
    priority: Optional[TaskPriority] = None
    status: Optional[TaskStatus] = None
    category_id: Optional[int] = None


class TaskResponse(BaseModel):
    """Схема ответа с задачей."""
    id: int
    title: str
    description: Optional[str]
    priority: str
    status: str
    user_id: int
    category_id: Optional[int]
    category_name: Optional[str] = None
    created_at: datetime
    updated_at: datetime
    
    class Config:
        orm_mode = True


class TaskListResponse(BaseModel):
    """Схема ответа со списком задач."""
    data: list[TaskResponse]
    total: int
    page: int
    limit: int
    pages: int
Файл app/repositories/base.py
python
from sqlalchemy.orm import Session
from typing import TypeVar, Generic, Type, List, Optional
from ..database import Base

ModelType = TypeVar("ModelType", bound=Base)


class BaseRepository(Generic[ModelType]):
    """Базовый репозиторий с общими CRUD операциями."""
    
    def __init__(self, model: Type[ModelType], db: Session):
        self.model = model
        self.db = db
    
    def get_by_id(self, id: int) -> Optional[ModelType]:
        return self.db.query(self.model).filter(self.model.id == id).first()
    
    def get_all(self, skip: int = 0, limit: int = 100) -> List[ModelType]:
        return self.db.query(self.model).offset(skip).limit(limit).all()
    
    def create(self, **kwargs) -> ModelType:
        instance = self.model(**kwargs)
        self.db.add(instance)
        self.db.commit()
        self.db.refresh(instance)
        return instance
    
    def update(self, id: int, **kwargs) -> Optional[ModelType]:
        instance = self.get_by_id(id)
        if instance:
            for key, value in kwargs.items():
                setattr(instance, key, value)
            self.db.commit()
            self.db.refresh(instance)
        return instance
    
    def delete(self, id: int) -> bool:
        instance = self.get_by_id(id)
        if instance:
            self.db.delete(instance)
            self.db.commit()
            return True
        return False
    
    def count(self) -> int:
        return self.db.query(self.model).count()
Файл app/repositories/user_repository.py
python
from sqlalchemy.orm import Session
from typing import Optional

from .base import BaseRepository
from ..models.user import User


class UserRepository(BaseRepository[User]):
    """Репозиторий для работы с пользователями."""
    
    def __init__(self, db: Session):
        super().__init__(User, db)
    
    def get_by_username(self, username: str) -> Optional[User]:
        return self.db.query(User).filter(User.username == username).first()
    
    def get_by_email(self, email: str) -> Optional[User]:
        return self.db.query(User).filter(User.email == email).first()
    
    def get_active_users(self, skip: int = 0, limit: int = 100):
        return self.db.query(User).filter(User.is_active == True).offset(skip).limit(limit).all()
Файл app/repositories/category_repository.py
python
from sqlalchemy.orm import Session
from typing import Optional

from .base import BaseRepository
from ..models.category import Category


class CategoryRepository(BaseRepository[Category]):
    """Репозиторий для работы с категориями."""
    
    def __init__(self, db: Session):
        super().__init__(Category, db)
    
    def get_by_name(self, name: str) -> Optional[Category]:
        return self.db.query(Category).filter(Category.name == name).first()
Файл app/repositories/task_repository.py
python
from sqlalchemy.orm import Session
from typing import Optional, List
from ..models.task import Task, TaskStatus, TaskPriority


class TaskRepository:
    """Репозиторий для работы с задачами."""
    
    def __init__(self, db: Session):
        self.db = db
    
    def get_by_id(self, task_id: int) -> Optional[Task]:
        return self.db.query(Task).filter(Task.id == task_id).first()
    
    def get_by_user(
        self,
        user_id: int,
        skip: int = 0,
        limit: int = 100,
        status: Optional[TaskStatus] = None,
        priority: Optional[TaskPriority] = None
    ) -> List[Task]:
        query = self.db.query(Task).filter(Task.user_id == user_id)
        
        if status:
            query = query.filter(Task.status == status)
        if priority:
            query = query.filter(Task.priority == priority)
        
        return query.order_by(Task.created_at.desc()).offset(skip).limit(limit).all()
    
    def create(self, user_id: int, **kwargs) -> Task:
        task = Task(user_id=user_id, **kwargs)
        self.db.add(task)
        self.db.commit()
        self.db.refresh(task)
        return task
    
    def update(self, task_id: int, **kwargs) -> Optional[Task]:
        task = self.get_by_id(task_id)
        if task:
            for key, value in kwargs.items():
                setattr(task, key, value)
            self.db.commit()
            self.db.refresh(task)
        return task
    
    def delete(self, task_id: int) -> bool:
        task = self.get_by_id(task_id)
        if task:
            self.db.delete(task)
            self.db.commit()
            return True
        return False
    
    def count_by_user(self, user_id: int) -> int:
        return self.db.query(Task).filter(Task.user_id == user_id).count()
    
    def get_completed_count(self, user_id: int) -> int:
        return self.db.query(Task).filter(
            Task.user_id == user_id,
            Task.status == TaskStatus.COMPLETED
        ).count()
Файл app/services/user_service.py
python
from sqlalchemy.orm import Session
from ..repositories.user_repository import UserRepository
from ..schemas.user import UserCreate, UserUpdate
from ..utils.auth import create_access_token
from ..utils.exceptions import BusinessError


class UserService:
    """Сервис для работы с пользователями."""
    
    def __init__(self, db: Session):
        self.db = db
        self.repo = UserRepository(db)
    
    def register(self, user_data: UserCreate):
        """Регистрация нового пользователя."""
        # Проверка уникальности
        if self.repo.get_by_username(user_data.username):
            raise BusinessError("Username already exists")
        if self.repo.get_by_email(user_data.email):
            raise BusinessError("Email already exists")
        
        # Создание пользователя
        user = self.repo.create(
            username=user_data.username,
            email=user_data.email,
            password_hash=""
        )
        user.set_password(user_data.password)
        self.db.commit()
        
        return user
    
    def authenticate(self, username: str, password: str):
        """Аутентификация пользователя."""
        user = self.repo.get_by_username(username)
        if not user:
            raise BusinessError("Invalid credentials")
        
        if not user.verify_password(password):
            raise BusinessError("Invalid credentials")
        
        if not user.is_active:
            raise BusinessError("Account is disabled")
        
        # Создание токена
        token = create_access_token(data={"sub": str(user.id), "username": user.username})
        
        return {"access_token": token, "user": user}
    
    def get_user(self, user_id: int):
        """Получение пользователя по ID."""
        user = self.repo.get_by_id(user_id)
        if not user:
            raise BusinessError("User not found")
        return user
    
    def update_user(self, user_id: int, user_data: UserUpdate):
        """Обновление пользователя."""
        user = self.get_user(user_id)
        
        update_data = user_data.dict(exclude_unset=True)
        
        if "password" in update_data:
            user.set_password(update_data.pop("password"))
        
        for key, value in update_data.items():
            setattr(user, key, value)
        
        self.db.commit()
        self.db.refresh(user)
        
        return user
    
    def delete_user(self, user_id: int):
        """Удаление пользователя."""
        user = self.get_user(user_id)
        self.db.delete(user)
        self.db.commit()
Файл app/services/task_service.py
python
from sqlalchemy.orm import Session
from typing import List, Optional
from ..repositories.task_repository import TaskRepository
from ..repositories.category_repository import CategoryRepository
from ..schemas.task import TaskCreate, TaskUpdate
from ..models.task import TaskStatus
from ..utils.exceptions import BusinessError


class TaskService:
    """Сервис для работы с задачами."""
    
    def __init__(self, db: Session):
        self.db = db
        self.repo = TaskRepository(db)
        self.category_repo = CategoryRepository(db)
    
    def create_task(self, user_id: int, task_data: TaskCreate):
        """Создание задачи."""
        # Проверка категории
        if task_data.category_id:
            category = self.category_repo.get_by_id(task_data.category_id)
            if not category:
                raise BusinessError("Category not found")
        
        return self.repo.create(
            user_id=user_id,
            title=task_data.title,
            description=task_data.description,
            priority=task_data.priority,
            category_id=task_data.category_id
        )
    
    def get_user_tasks(
        self,
        user_id: int,
        skip: int = 0,
        limit: int = 20,
        status: Optional[str] = None,
        priority: Optional[str] = None
    ):
        """Получение задач пользователя."""
        tasks = self.repo.get_by_user(user_id, skip, limit, status, priority)
        
        # Обогащаем данными о категориях
        result = []
        for task in tasks:
            task_dict = {
                "id": task.id,
                "title": task.title,
                "description": task.description,
                "priority": task.priority,
                "status": task.status,
                "user_id": task.user_id,
                "category_id": task.category_id,
                "category_name": task.category.name if task.category else None,
                "created_at": task.created_at,
                "updated_at": task.updated_at
            }
            result.append(task_dict)
        
        total = self.repo.count_by_user(user_id)
        
        return {
            "data": result,
            "total": total,
            "page": skip // limit + 1 if limit else 1,
            "limit": limit,
            "pages": (total + limit - 1) // limit if limit else 1
        }
    
    def get_task(self, task_id: int, user_id: int):
        """Получение задачи по ID."""
        task = self.repo.get_by_id(task_id)
        
        if not task:
            raise BusinessError("Task not found")
        
        if task.user_id != user_id:
            raise BusinessError("Access denied")
        
        return task
    
    def update_task(self, task_id: int, user_id: int, task_data: TaskUpdate):
        """Обновление задачи."""
        task = self.get_task(task_id, user_id)
        
        update_data = task_data.dict(exclude_unset=True)
        
        # Проверка категории
        if "category_id" in update_data and update_data["category_id"]:
            category = self.category_repo.get_by_id(update_data["category_id"])
            if not category:
                raise BusinessError("Category not found")
        
        for key, value in update_data.items():
            setattr(task, key, value)
        
        self.db.commit()
        self.db.refresh(task)
        
        return task
    
    def delete_task(self, task_id: int, user_id: int):
        """Удаление задачи."""
        task = self.get_task(task_id, user_id)
        self.db.delete(task)
        self.db.commit()
    
    def complete_task(self, task_id: int, user_id: int):
        """Отметка задачи как выполненной."""
        task = self.get_task(task_id, user_id)
        task.status = TaskStatus.COMPLETED
        self.db.commit()
        self.db.refresh(task)
        return task
    
    def get_statistics(self, user_id: int):
        """Статистика по задачам пользователя."""
        total = self.repo.count_by_user(user_id)
        completed = self.repo.get_completed_count(user_id)
        
        return {
            "total": total,
            "completed": completed,
            "pending": total - completed,
            "completion_rate": round(completed / total * 100, 1) if total > 0 else 0
        }
Файл app/utils/exceptions.py
python
from fastapi import HTTPException, status


class BusinessError(HTTPException):
    """Исключение для бизнес-ошибок."""
    
    def __init__(self, detail: str):
        super().__init__(status_code=status.HTTP_400_BAD_REQUEST, detail=detail)


class NotFoundError(HTTPException):
    """Исключение для отсутствующих ресурсов."""
    
    def __init__(self, resource: str):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"{resource} not found"
        )


class UnauthorizedError(HTTPException):
    """Исключение для ошибок авторизации."""
    
    def __init__(self):
        super().__init__(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Unauthorized",
            headers={"WWW-Authenticate": "Bearer"}
        )
Файл app/utils/auth.py
python
from datetime import datetime, timedelta
from jose import JWTError, jwt
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

from ..config import settings
from .exceptions import UnauthorizedError

security = HTTPBearer()


def create_access_token(data: dict, expires_delta: timedelta = None) -> str:
    """Создание JWT токена."""
    to_encode = data.copy()
    
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)


def decode_token(token: str) -> dict:
    """Декодирование JWT токена."""
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        return payload
    except JWTError:
        raise UnauthorizedError()


async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> dict:
    """Получение текущего пользователя из токена."""
    token = credentials.credentials
    payload = decode_token(token)
    
    user_id = payload.get("sub")
    if not user_id:
        raise UnauthorizedError()
    
    return {"id": int(user_id), "username": payload.get("username")}
Файл app/api/tasks.py
python
from fastapi import APIRouter, Depends, Query
from sqlalchemy.orm import Session
from typing import Optional

from ..database import get_db
from ..services.task_service import TaskService
from ..schemas.task import TaskCreate, TaskUpdate, TaskResponse, TaskListResponse
from ..utils.auth import get_current_user
from ..models.task import TaskStatus, TaskPriority

router = APIRouter(prefix="/tasks", tags=["Tasks"])


@router.post("/", response_model=TaskResponse, status_code=201)
def create_task(
    task_data: TaskCreate,
    db: Session = Depends(get_db),
    current_user: dict = Depends(get_current_user)
):
    """Создание новой задачи."""
    service = TaskService(db)
    return service.create_task(current_user["id"], task_data)


@router.get("/", response_model=TaskListResponse)
def get_tasks(
    page: int = Query(1, ge=1),
    limit: int = Query(20, ge=1, le=100),
    status: Optional[TaskStatus] = None,
    priority: Optional[TaskPriority] = None,
    db: Session = Depends(get_db),
    current_user: dict = Depends(get_current_user)
):
    """Получение списка задач пользователя."""
    service = TaskService(db)
    skip = (page - 1) * limit
    return service.get_user_tasks(
        user_id=current_user["id"],
        skip=skip,
        limit=limit,
        status=status,
        priority=priority
    )


@router.get("/{task_id}", response_model=TaskResponse)
def get_task(
    task_id: int,
    db: Session = Depends(get_db),
    current_user: dict = Depends(get_current_user)
):
    """Получение задачи по ID."""
    service = TaskService(db)
    return service.get_task(task_id, current_user["id"])


@router.put("/{task_id}", response_model=TaskResponse)
def update_task(
    task_id: int,
    task_data: TaskUpdate,
    db: Session = Depends(get_db),
    current_user: dict = Depends(get_current_user)
):
    """Обновление задачи."""
    service = TaskService(db)
    return service.update_task(task_id, current_user["id"], task_data)


@router.delete("/{task_id}", status_code=204)
def delete_task(
    task_id: int,
    db: Session = Depends(get_db),
    current_user: dict = Depends(get_current_user)
):
    """Удаление задачи."""
    service = TaskService(db)
    service.delete_task(task_id, current_user["id"])


@router.post("/{task_id}/complete", response_model=TaskResponse)
def complete_task(
    task_id: int,
    db: Session = Depends(get_db),
    current_user: dict = Depends(get_current_user)
):
    """Отметка задачи как выполненной."""
    service = TaskService(db)
    return service.complete_task(task_id, current_user["id"])


@router.get("/stats/summary")
def get_stats(
    db: Session = Depends(get_db),
    current_user: dict = Depends(get_current_user)
):
    """Статистика по задачам пользователя."""
    service = TaskService(db)
    return service.get_statistics(current_user["id"])
Файл app/api/users.py
python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session

from ..database import get_db
from ..services.user_service import UserService
from ..schemas.user import UserCreate, UserResponse, UserLogin, TokenResponse
from ..utils.auth import get_current_user

router = APIRouter(prefix="/users", tags=["Users"])


@router.post("/register", response_model=UserResponse, status_code=201)
def register(user_data: UserCreate, db: Session = Depends(get_db)):
    """Регистрация нового пользователя."""
    service = UserService(db)
    return service.register(user_data)


@router.post("/login", response_model=TokenResponse)
def login(user_data: UserLogin, db: Session = Depends(get_db)):
    """Вход пользователя."""
    service = UserService(db)
    result = service.authenticate(user_data.username, user_data.password)
    
    return TokenResponse(access_token=result["access_token"])


@router.get("/me", response_model=UserResponse)
def get_current_user_info(
    current_user: dict = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Получение информации о текущем пользователе."""
    service = UserService(db)
    return service.get_user(current_user["id"])
Файл app/main.py
python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager

from .database import engine, Base
from .config import settings
from .api import users, tasks

# Создание таблиц
Base.metadata.create_all(bind=engine)


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Управление жизненным циклом приложения."""
    print("🚀 Starting application...")
    yield
    print("🛑 Shutting down...")


app = FastAPI(
    title=settings.APP_NAME,
    version=settings.APP_VERSION,
    lifespan=lifespan
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# Регистрация роутеров
app.include_router(users.router)
app.include_router(tasks.router)


@app.get("/")
def root():
    return {
        "message": f"Welcome to {settings.APP_NAME}",
        "version": settings.APP_VERSION,
        "docs": "/docs",
        "redoc": "/redoc"
    }


@app.get("/health")
def health_check():
    return {"status": "healthy"}
Файл requirements.txt
txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlalchemy==2.0.23
pydantic==2.5.0
python-dotenv==1.0.0
passlib[bcrypt]==1.7.4
python-jose[cryptography]==3.3.0
Файл .env
env
APP_NAME=Task Manager API
APP_VERSION=1.0.0
DEBUG=True

DATABASE_URL=sqlite:///./task_manager.db

SECRET_KEY=your-secret-key-change-in-production
ACCESS_TOKEN_EXPIRE_MINUTES=30

DEFAULT_PAGE_SIZE=20
Запуск и тестирование
bash
# Установка зависимостей
pip install -r requirements.txt

# Запуск сервера
uvicorn app.main:app --reload

# Документация
# http://localhost:8000/docs
# http://localhost:8000/redoc
Примеры запросов
bash
# Регистрация
curl -X POST http://localhost:8000/users/register \
  -H "Content-Type: application/json" \
  -d '{"username":"ivan","email":"ivan@example.com","password":"secret123"}'

# Логин
curl -X POST http://localhost:8000/users/login \
  -H "Content-Type: application/json" \
  -d '{"username":"ivan","password":"secret123"}'

# Создание задачи (с токеном)
TOKEN="your-token"
curl -X POST http://localhost:8000/tasks/ \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Изучить FastAPI","priority":"high"}'

# Получение задач
curl -X GET "http://localhost:8000/tasks/?page=1&limit=10" \
  -H "Authorization: Bearer $TOKEN"
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (CRUD для одной сущности)

Варианты 9-17: средний (2-3 сущности, связи)

Варианты 18-25: сложный (аутентификация, роли, статистика)

Варианты 1-8 (Базовый уровень)
№	Сущность	Поля	Особенности
1	Книга (Book)	title, author, year, isbn	CRUD
2	Фильм (Movie)	title, director, year, rating	CRUD
3	Продукт (Product)	name, price, stock	CRUD
4	Студент (Student)	name, email, course	CRUD
5	Заметка (Note)	title, content, tags	CRUD
6	Событие (Event)	name, date, location	CRUD
7	Контакт (Contact)	name, phone, email	CRUD
8	Задача (Task)	title, completed	CRUD
Варианты 9-17 (Средний уровень)
№	Сущности	Связи	Дополнительно
9	User, Post	1:N	Комментарии
10	Author, Book	1:N	Категории
11	Category, Product	1:N	Поиск
12	Customer, Order	1:N	Пагинация
13	Doctor, Appointment	1:N	Фильтрация
14	Playlist, Song	N:M	Сортировка
15	Team, Player	1:N	Статистика
16	Course, Student	N:M	Оценки
17	Project, Task	1:N	Статусы
Варианты 18-25 (Сложный уровень)
№	Тема	Компоненты
18	Блог	User, Post, Comment, Category
19	Магазин	User, Product, Order, Cart
20	Библиотека	User, Book, Loan, Author
21	Образование	Student, Course, Enrollment, Grade
22	CRM	User, Customer, Deal, Activity
23	Доставка	User, Order, Courier, Delivery
24	Соцсеть	User, Post, Like, Friend
25	Авиабилеты	User, Flight, Booking, Seat
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Сервис не работает или содержит критические ошибки
3 (удовлетворительно)	Базовый CRUD, нет валидации или обработки ошибок
4 (хорошо)	CRUD + валидация + обработка ошибок
5 (отлично)	Полная архитектура + аутентификация + документация
5. Шпаргалка
python
# === СТРУКТУРА ===
app/
├── main.py          # FastAPI приложение
├── config.py        # Настройки
├── database.py      # Подключение к БД
├── models/          # SQLAlchemy модели
├── schemas/         # Pydantic схемы
├── repositories/    # CRUD операции
├── services/        # Бизнес-логика
├── api/             # Роутеры
└── utils/           # Утилиты

# === КЛЮЧЕВЫЕ МОМЕНТЫ ===
# Разделение ответственности
# Валидация через Pydantic
# Dependency Injection
# Обработка ошибок
# Документация (автоматическая)
Карточка студента
text
ПЗ 2.48. РЕАЛИЗАЦИЯ ПОЛНОЦЕННОГО ВЕБ-СЕРВИСА

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== СУЩНОСТИ ===

1. _____________
2. _____________
3. _____________

=== КОМПОНЕНТЫ ===

□ Модели (SQLAlchemy)
□ Схемы (Pydantic)
□ Репозитории (CRUD)
□ Сервисы (бизнес-логика)
□ API роутеры
□ Аутентификация (JWT)
□ Валидация
□ Обработка ошибок
□ Документация (/docs)

=== ОТЧЁТ ===

Ссылка на репозиторий: _____________
Документация API: _____________
Тесты: _____

Дата выполнения: _____________
