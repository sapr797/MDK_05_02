# ПЗ 2.30. Интеграция базы данных с FastAPI

**Тема:** Веб-фреймворки, работа с базами данных, SQLAlchemy, Dependency Injection, асинхронность

**Цель работы:**  
Научиться интегрировать базу данных с FastAPI, использовать SQLAlchemy для асинхронных операций, правильно организовывать dependency injection для сессий БД, создавать полноценные API-эндпоинты с сохранением данных.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.9+
- Установленные пакеты: `fastapi`, `uvicorn`, `sqlalchemy`, `alembic`, `pydantic`

```bash
pip install fastapi uvicorn sqlalchemy alembic pydantic
Главная мысль: FastAPI + SQLAlchemy — идеальная пара для создания надёжных API. Dependency Injection делает код чистым и тестируемым.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Архитектура приложения
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                          АРХИТЕКТУРА ПРИЛОЖЕНИЯ                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐│
│  │   Клиент    │────►│   FastAPI   │────►│  SQLAlchemy │────►│  База       ││
│  │  (браузер)  │◄────│  (API)      │◄────│  (ORM)      │◄────│  данных     ││
│  └─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘│
│                             │                                              │
│                             ▼                                              │
│                      ┌─────────────┐                                       │
│                      │  Pydantic   │                                       │
│                      │ (валидация) │                                       │
│                      └─────────────┘                                       │
└─────────────────────────────────────────────────────────────────────────────┘
1.2. Компоненты интеграции
Компонент	Назначение
Engine	Подключение к БД (один на приложение)
SessionLocal	Фабрика сессий
Dependency	Функция для получения сессии в эндпоинтах
Models	SQLAlchemy модели (таблицы)
Schemas	Pydantic схемы (валидация входящих/исходящих данных)
CRUD	Функции для работы с БД
1.3. Синхронный vs Асинхронный доступ
Характеристика	Синхронный	Асинхронный (2.0)
Производительность	Ниже	Выше
Сложность	Проще	Сложнее
Поддержка	Полная	Экспериментальная
Драйвер	psycopg2	asyncpg, aiosqlite
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая интеграцию базы данных с FastAPI.

Техническое задание (нулевой вариант)
Разработайте REST API для управления задачами (To-Do) с интеграцией базы данных SQLite через SQLAlchemy. API должно поддерживать:

GET /todos — получение списка задач (с пагинацией)

GET /todos/{id} — получение задачи по ID

POST /todos — создание задачи

PUT /todos/{id} — полное обновление задачи

PATCH /todos/{id} — частичное обновление задачи

DELETE /todos/{id} — удаление задачи

GET /todos/search — поиск по названию

Эталонная реализация
python
#!/usr/bin/env python3
"""
main.py — FastAPI приложение с интеграцией базы данных.
"""

from fastapi import FastAPI, Depends, HTTPException, Query, status
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy import create_engine, Column, Integer, String, Boolean, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from pydantic import BaseModel, Field
from datetime import datetime
from typing import List, Optional

# ============================================================
# НАСТРОЙКА БАЗЫ ДАННЫХ
# ============================================================

DATABASE_URL = "sqlite:///./todos.db"

engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False}  # для SQLite
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()


# ============================================================
# МОДЕЛЬ SQLALCHEMY
# ============================================================

class TodoDB(Base):
    __tablename__ = "todos"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False, index=True)
    description = Column(String(1000), nullable=True)
    completed = Column(Boolean, default=False)
    priority = Column(Integer, default=1)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)


# ============================================================
# PYDANTIC СХЕМЫ
# ============================================================

class TodoCreate(BaseModel):
    """Схема для создания задачи."""
    title: str = Field(..., min_length=1, max_length=200)
    description: Optional[str] = Field(None, max_length=1000)
    priority: int = Field(1, ge=1, le=5)


class TodoUpdate(BaseModel):
    """Схема для полного обновления задачи."""
    title: str = Field(..., min_length=1, max_length=200)
    description: Optional[str] = Field(None, max_length=1000)
    completed: bool
    priority: int = Field(..., ge=1, le=5)


class TodoPatch(BaseModel):
    """Схема для частичного обновления задачи."""
    title: Optional[str] = Field(None, min_length=1, max_length=200)
    description: Optional[str] = Field(None, max_length=1000)
    completed: Optional[bool] = None
    priority: Optional[int] = Field(None, ge=1, le=5)


class TodoResponse(BaseModel):
    """Схема для ответа."""
    id: int
    title: str
    description: Optional[str] = None
    completed: bool
    priority: int
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        from_attributes = True


class TodoListResponse(BaseModel):
    """Схема для списка задач с пагинацией."""
    data: List[TodoResponse]
    total: int
    page: int
    limit: int
    pages: int


# ============================================================
# ФУНКЦИЯ ЗАВИСИМОСТИ ДЛЯ ПОЛУЧЕНИЯ СЕССИИ
# ============================================================

def get_db():
    """
    Dependency Injection для получения сессии базы данных.
    Сессия автоматически закрывается после обработки запроса.
    """
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


# ============================================================
# CRUD ФУНКЦИИ
# ============================================================

class TodoCRUD:
    """CRUD операции для задач."""
    
    @staticmethod
    def get_all(
        db: Session,
        skip: int = 0,
        limit: int = 100,
        completed: Optional[bool] = None,
        min_priority: Optional[int] = None
    ) -> tuple[List[TodoDB], int]:
        """
        Получение всех задач с пагинацией и фильтрацией.
        
        Returns:
            tuple: (список задач, общее количество)
        """
        query = db.query(TodoDB)
        
        if completed is not None:
            query = query.filter(TodoDB.completed == completed)
        
        if min_priority is not None:
            query = query.filter(TodoDB.priority >= min_priority)
        
        total = query.count()
        todos = query.order_by(TodoDB.priority.desc(), TodoDB.created_at.desc()).offset(skip).limit(limit).all()
        
        return todos, total
    
    @staticmethod
    def get_by_id(db: Session, todo_id: int) -> Optional[TodoDB]:
        """Получение задачи по ID."""
        return db.query(TodoDB).filter(TodoDB.id == todo_id).first()
    
    @staticmethod
    def create(db: Session, todo: TodoCreate) -> TodoDB:
        """Создание новой задачи."""
        db_todo = TodoDB(
            title=todo.title,
            description=todo.description,
            priority=todo.priority
        )
        db.add(db_todo)
        db.commit()
        db.refresh(db_todo)
        return db_todo
    
    @staticmethod
    def update(db: Session, todo_id: int, todo: TodoUpdate) -> Optional[TodoDB]:
        """Полное обновление задачи."""
        db_todo = TodoCRUD.get_by_id(db, todo_id)
        if not db_todo:
            return None
        
        db_todo.title = todo.title
        db_todo.description = todo.description
        db_todo.completed = todo.completed
        db_todo.priority = todo.priority
        db_todo.updated_at = datetime.utcnow()
        
        db.commit()
        db.refresh(db_todo)
        return db_todo
    
    @staticmethod
    def patch(db: Session, todo_id: int, todo: TodoPatch) -> Optional[TodoDB]:
        """Частичное обновление задачи."""
        db_todo = TodoCRUD.get_by_id(db, todo_id)
        if not db_todo:
            return None
        
        update_data = todo.model_dump(exclude_unset=True)
        for key, value in update_data.items():
            setattr(db_todo, key, value)
        
        db_todo.updated_at = datetime.utcnow()
        
        db.commit()
        db.refresh(db_todo)
        return db_todo
    
    @staticmethod
    def delete(db: Session, todo_id: int) -> bool:
        """Удаление задачи."""
        db_todo = TodoCRUD.get_by_id(db, todo_id)
        if not db_todo:
            return False
        
        db.delete(db_todo)
        db.commit()
        return True
    
    @staticmethod
    def search(db: Session, query: str, skip: int = 0, limit: int = 100) -> tuple[List[TodoDB], int]:
        """Поиск задач по названию."""
        search_query = db.query(TodoDB).filter(TodoDB.title.contains(query))
        total = search_query.count()
        todos = search_query.offset(skip).limit(limit).all()
        return todos, total


# ============================================================
# СОЗДАНИЕ ТАБЛИЦ
# ============================================================

Base.metadata.create_all(bind=engine)


# ============================================================
# FASTAPI ПРИЛОЖЕНИЕ
# ============================================================

app = FastAPI(
    title="Todo API",
    description="REST API для управления задачами с интеграцией БД",
    version="1.0.0"
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)


# ============================================================
# API ЭНДПОИНТЫ
# ============================================================

@app.get("/")
async def root():
    """Корневой эндпоинт."""
    return {
        "message": "Todo API",
        "endpoints": {
            "GET /todos": "Список задач",
            "GET /todos/{id}": "Задача по ID",
            "POST /todos": "Создать задачу",
            "PUT /todos/{id}": "Полное обновление",
            "PATCH /todos/{id}": "Частичное обновление",
            "DELETE /todos/{id}": "Удалить задачу",
            "GET /todos/search": "Поиск"
        }
    }


@app.get("/todos", response_model=TodoListResponse)
async def get_todos(
    page: int = Query(1, ge=1, description="Номер страницы"),
    limit: int = Query(10, ge=1, le=100, description="Записей на странице"),
    completed: Optional[bool] = Query(None, description="Фильтр по статусу"),
    min_priority: Optional[int] = Query(None, ge=1, le=5, description="Минимальный приоритет"),
    db: Session = Depends(get_db)
):
    """
    Получение списка задач с пагинацией и фильтрацией.
    """
    skip = (page - 1) * limit
    todos, total = TodoCRUD.get_all(
        db,
        skip=skip,
        limit=limit,
        completed=completed,
        min_priority=min_priority
    )
    
    pages = (total + limit - 1) // limit if total > 0 else 1
    
    return TodoListResponse(
        data=[TodoResponse.model_validate(todo) for todo in todos],
        total=total,
        page=page,
        limit=limit,
        pages=pages
    )


@app.get("/todos/{todo_id}", response_model=TodoResponse)
async def get_todo(
    todo_id: int,
    db: Session = Depends(get_db)
):
    """
    Получение задачи по ID.
    """
    todo = TodoCRUD.get_by_id(db, todo_id)
    if not todo:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Задача с ID {todo_id} не найдена"
        )
    return TodoResponse.model_validate(todo)


@app.post("/todos", response_model=TodoResponse, status_code=status.HTTP_201_CREATED)
async def create_todo(
    todo: TodoCreate,
    db: Session = Depends(get_db)
):
    """
    Создание новой задачи.
    """
    created = TodoCRUD.create(db, todo)
    return TodoResponse.model_validate(created)


@app.put("/todos/{todo_id}", response_model=TodoResponse)
async def update_todo(
    todo_id: int,
    todo: TodoUpdate,
    db: Session = Depends(get_db)
):
    """
    Полное обновление задачи.
    """
    updated = TodoCRUD.update(db, todo_id, todo)
    if not updated:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Задача с ID {todo_id} не найдена"
        )
    return TodoResponse.model_validate(updated)


@app.patch("/todos/{todo_id}", response_model=TodoResponse)
async def patch_todo(
    todo_id: int,
    todo: TodoPatch,
    db: Session = Depends(get_db)
):
    """
    Частичное обновление задачи.
    """
    updated = TodoCRUD.patch(db, todo_id, todo)
    if not updated:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Задача с ID {todo_id} не найдена"
        )
    return TodoResponse.model_validate(updated)


@app.delete("/todos/{todo_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_todo(
    todo_id: int,
    db: Session = Depends(get_db)
):
    """
    Удаление задачи.
    """
    deleted = TodoCRUD.delete(db, todo_id)
    if not deleted:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Задача с ID {todo_id} не найдена"
        )
    return None


@app.get("/todos/search", response_model=TodoListResponse)
async def search_todos(
    q: str = Query(..., min_length=1, description="Поисковый запрос"),
    page: int = Query(1, ge=1),
    limit: int = Query(10, ge=1, le=100),
    db: Session = Depends(get_db)
):
    """
    Поиск задач по названию.
    """
    skip = (page - 1) * limit
    todos, total = TodoCRUD.search(db, q, skip=skip, limit=limit)
    
    pages = (total + limit - 1) // limit if total > 0 else 1
    
    return TodoListResponse(
        data=[TodoResponse.model_validate(todo) for todo in todos],
        total=total,
        page=page,
        limit=limit,
        pages=pages
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
bash
# Создание задачи
curl -X POST http://localhost:8000/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "Изучить FastAPI", "description": "Понять интеграцию с БД", "priority": 5}'

# Получение списка задач
curl -X GET "http://localhost:8000/todos?page=1&limit=10"

# Получение задачи по ID
curl -X GET http://localhost:8000/todos/1

# Частичное обновление
curl -X PATCH http://localhost:8000/todos/1 \
  -H "Content-Type: application/json" \
  -d '{"completed": true}'

# Поиск
curl -X GET "http://localhost:8000/todos/search?q=FastAPI"

# Удаление
curl -X DELETE http://localhost:8000/todos/1
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (одна модель, простые CRUD)

Варианты 9-17: средний (две модели, связи)

Варианты 18-25: сложный (3+ модели, сложные связи)

Варианты 1-8 (Базовый уровень)
№	Модель	Поля	Особенности
1	User	id, name, email, age	CRUD
2	Product	id, name, price, stock	CRUD
3	Post	id, title, content	CRUD
4	Task	id, title, completed	CRUD
5	Book	id, title, author, year	CRUD + поиск
6	Movie	id, title, director, rating	CRUD + фильтрация
7	Student	id, name, group, score	CRUD
8	Employee	id, name, position, salary	CRUD
Варианты 9-17 (Средний уровень)
№	Модели	Связи	Дополнительно
9	User, Post	1:N	GET /users/{id}/posts
10	Author, Book	1:N	GET /authors/{id}/books
11	Category, Product	1:N	GET /categories/{id}/products
12	Customer, Order	1:N	GET /customers/{id}/orders
13	Doctor, Appointment	1:N	GET /doctors/{id}/appointments
14	Team, Player	1:N	GET /teams/{id}/players
15	Playlist, Song	N:M	GET /playlists/{id}/songs
16	Project, Task	1:N	GET /projects/{id}/tasks
17	Course, Student	N:M	POST /courses/{id}/enroll
Варианты 18-25 (Сложный уровень)
№	Тема	Модели	Особенности
18	Блог	User, Post, Comment	Полноценный блог
19	Магазин	User, Product, Order, OrderItem	Корзина, заказы
20	Библиотека	User, Book, Loan, Author	Выдача книг
21	Образование	Student, Course, Enrollment, Grade	Успеваемость
22	CRM	User, Customer, Deal, Activity	Воронка продаж
23	Доставка	User, Order, Courier, Delivery	Трекинг
24	Соцсеть	User, Post, Like, Friend	Лента, друзья
25	Авиабилеты	User, Flight, Booking, Seat	Бронирование
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	API не работает или БД не интегрирована
3 (удовлетворительно)	Работают базовые CRUD операции
4 (хорошо)	+ пагинация, фильтрация, поиск
5 (отлично)	+ валидация, обработка ошибок, документация
5. Шпаргалка
python
# === ПОДКЛЮЧЕНИЕ К БД ===
DATABASE_URL = "sqlite:///./app.db"
engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# === DEPENDENCY ===
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# === МОДЕЛЬ ===
class ModelDB(Base):
    __tablename__ = "table_name"
    id = Column(Integer, primary_key=True, index=True)

# === SCHEMA ===
class ModelCreate(BaseModel):
    name: str

class ModelResponse(ModelCreate):
    id: int
    class Config:
        from_attributes = True

# === CRUD ===
def get_all(db: Session, skip: int = 0, limit: int = 100):
    return db.query(ModelDB).offset(skip).limit(limit).all()

def get_by_id(db: Session, id: int):
    return db.query(ModelDB).filter(ModelDB.id == id).first()

def create(db: Session, data: ModelCreate):
    db_item = ModelDB(**data.model_dump())
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item

# === ЭНДПОИНТ ===
@app.get("/items", response_model=List[ModelResponse])
def get_items(db: Session = Depends(get_db)):
    return get_all(db)
Карточка студента
text
ПЗ 2.30. ИНТЕГРАЦИЯ БАЗЫ ДАННЫХ С FASTAPI

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== МОДЕЛИ ===

1. _____________
2. _____________
3. _____________

=== API ЭНДПОИНТЫ ===

□ GET /resource
□ GET /resource/{id}
□ POST /resource
□ PUT /resource/{id}
□ PATCH /resource/{id}
□ DELETE /resource/{id}
□ GET /resource/search
□ Связанные эндпоинты

=== ОТЧЁТ ===

Ссылка на репозиторий: _____________
Скриншоты тестирования: _____________
Swagger документация: _____________

Дата выполнения: _____________
