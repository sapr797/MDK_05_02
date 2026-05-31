# Тема 2.18. Маршруты, обработчики, запросы и ответы в FastAPI

**Цель лекции:**  
Изучить создание маршрутов и обработчиков в FastAPI, освоить работу с различными типами запросов и ответов, научиться обрабатывать параметры пути, query-параметры, тело запроса, заголовки и файлы.

> Главная мысль: **FastAPI превращает аннотации типов в мощный инструмент валидации, документации и автодополнения.**

---

## Содержание

1. [Основы маршрутизации](#1-основы-маршрутизации)
2. [Параметры запроса](#2-параметры-запроса)
3. [Тело запроса](#3-тело-запроса)
4. [Ответы](#4-ответы)
5. [Обработка ошибок](#5-обработка-ошибок)
6. [Зависимости (Dependency Injection)](#6-зависимости-dependency-injection)
7. [Практические примеры](#7-практические-примеры)
8. [Контрольные вопросы](#8-контрольные-вопросы)
9. [Практическое задание](#9-практическое-задание)
10. [Шпаргалка](#10-шпаргалка)

---

## 1. Основы маршрутизации

### 1.1. Декораторы маршрутов

```python
from fastapi import FastAPI

app = FastAPI()

# GET маршрут
@app.get('/')
def root():
    return {"message": "Hello World"}

# POST маршрут
@app.post('/items')
def create_item():
    return {"message": "Item created"}

# PUT маршрут
@app.put('/items/{item_id}')
def update_item(item_id: int):
    return {"message": f"Item {item_id} updated"}

# DELETE маршрут
@app.delete('/items/{item_id}')
def delete_item(item_id: int):
    return {"message": f"Item {item_id} deleted"}

# PATCH маршрут
@app.patch('/items/{item_id}')
def patch_item(item_id: int):
    return {"message": f"Item {item_id} partially updated"}
1.2. Параметры пути (Path Parameters)
python
from fastapi import FastAPI

app = FastAPI()

# Простой параметр
@app.get('/users/{user_id}')
def get_user(user_id: int):
    return {"user_id": user_id}

# Несколько параметров
@app.get('/users/{user_id}/items/{item_id}')
def get_user_item(user_id: int, item_id: int):
    return {"user_id": user_id, "item_id": item_id}

# Параметры со строками
@app.get('/products/{product_slug}')
def get_product(product_slug: str):
    return {"product": product_slug}

# Валидация параметров пути с помощью Path
from fastapi import Path

@app.get('/items/{item_id}')
def get_item(
    item_id: int = Path(..., gt=0, le=1000, description="ID товара")
):
    return {"item_id": item_id}
1.3. Query-параметры
python
from fastapi import FastAPI, Query
from typing import Optional, List

app = FastAPI()

# Базовые query-параметры
@app.get('/items')
def get_items(
    skip: int = 0,
    limit: int = 10,
    sort: str = "asc"
):
    return {"skip": skip, "limit": limit, "sort": sort}

# Опциональные параметры
@app.get('/users')
def get_users(
    name: Optional[str] = None,
    age: Optional[int] = None
):
    result = {"users": []}
    if name:
        result["filter"] = f"name={name}"
    if age:
        result["filter"] = f"age={age}"
    return result

# Валидация query-параметров
@app.get('/products')
def get_products(
    page: int = Query(1, ge=1, description="Номер страницы"),
    page_size: int = Query(10, ge=1, le=100, description="Размер страницы"),
    category: Optional[str] = Query(None, min_length=3, max_length=50),
    tags: List[str] = Query([], description="Список тегов")
):
    return {
        "page": page,
        "page_size": page_size,
        "category": category,
        "tags": tags
    }

# Обязательный query-параметр
@app.get('/search')
def search(q: str = Query(..., min_length=1)):
    return {"query": q}
1.4. Комбинация параметров пути и query
python
@app.get('/users/{user_id}/orders')
def get_user_orders(
    user_id: int,                      # path parameter
    status: Optional[str] = None,      # query parameter
    limit: int = 10,                   # query parameter
    offset: int = 0                    # query parameter
):
    return {
        "user_id": user_id,
        "status": status,
        "limit": limit,
        "offset": offset
    }
2. Параметры запроса
2.1. Тело запроса с Pydantic
python
from fastapi import FastAPI
from pydantic import BaseModel, Field
from typing import Optional, List
from datetime import datetime

app = FastAPI()

# Модель для тела запроса
class ItemCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    price: float = Field(..., gt=0, le=1000000)
    description: Optional[str] = Field(None, max_length=500)
    tags: List[str] = []
    created_at: Optional[datetime] = None

class ItemResponse(ItemCreate):
    id: int
    created_at: datetime

# Использование модели
@app.post('/items', response_model=ItemResponse, status_code=201)
def create_item(item: ItemCreate):
    # item автоматически валидируется
    return {
        "id": 1,
        "name": item.name,
        "price": item.price,
        "description": item.description,
        "tags": item.tags,
        "created_at": datetime.now()
    }
2.2. Несколько моделей в одном запросе
python
class User(BaseModel):
    name: str
    email: str

class Order(BaseModel):
    product: str
    quantity: int

@app.post('/users/{user_id}/orders')
def create_user_order(
    user_id: int,
    user: User,
    order: Order,
    priority: int = 1
):
    return {
        "user_id": user_id,
        "user": user.dict(),
        "order": order.dict(),
        "priority": priority
    }
2.3. Заголовки запроса (Headers)
python
from fastapi import Header, HTTPException

@app.get('/protected')
def protected_route(
    authorization: str = Header(..., description="Bearer token"),
    user_agent: Optional[str] = Header(None),
    x_request_id: Optional[str] = Header(None, alias="X-Request-ID")
):
    if not authorization.startswith('Bearer '):
        raise HTTPException(status_code=401, detail="Invalid token")
    
    return {
        "token": authorization,
        "user_agent": user_agent,
        "request_id": x_request_id
    }
2.4. Cookies
python
from fastapi import Cookie, Response

@app.get('/set-cookie')
def set_cookie(response: Response):
    response.set_cookie(key="session_id", value="abc123", max_age=3600)
    response.set_cookie(key="user_pref", value="dark_mode")
    return {"message": "Cookies set"}

@app.get('/get-cookie')
def get_cookie(
    session_id: Optional[str] = Cookie(None),
    user_pref: Optional[str] = Cookie(None)
):
    return {
        "session_id": session_id,
        "user_pref": user_pref
    }
2.5. Загрузка файлов
python
from fastapi import File, UploadFile
from typing import List
import shutil
from pathlib import Path

# Одиночный файл
@app.post('/upload')
def upload_file(file: UploadFile = File(...)):
    # Сохраняем файл
    file_path = Path(f"uploads/{file.filename}")
    file_path.parent.mkdir(exist_ok=True)
    
    with open(file_path, 'wb') as buffer:
        shutil.copyfileobj(file.file, buffer)
    
    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": file_path.stat().st_size
    }

# Несколько файлов
@app.post('/upload-multiple')
def upload_multiple(files: List[UploadFile] = File(...)):
    results = []
    for file in files:
        file_path = Path(f"uploads/{file.filename}")
        with open(file_path, 'wb') as buffer:
            shutil.copyfileobj(file.file, buffer)
        results.append({
            "filename": file.filename,
            "size": file_path.stat().st_size
        })
    return {"files": results}
2.6. Формы (Form Data)
python
from fastapi import Form

@app.post('/login')
def login(
    username: str = Form(...),
    password: str = Form(...)
):
    # Здесь должна быть проверка логина
    return {"username": username, "status": "logged_in"}

@app.post('/register')
def register(
    username: str = Form(..., min_length=3, max_length=50),
    email: str = Form(...),
    password: str = Form(..., min_length=8),
    age: int = Form(..., ge=18, le=120)
):
    return {
        "username": username,
        "email": email,
        "age": age
    }
3. Тело запроса
3.1. Разные типы ответов
python
from fastapi.responses import (
    JSONResponse,
    HTMLResponse,
    PlainTextResponse,
    FileResponse,
    RedirectResponse,
    StreamingResponse
)
import json

# JSON ответ (по умолчанию)
@app.get('/json')
def json_response():
    return {"message": "This is JSON"}

# HTML ответ
@app.get('/html', response_class=HTMLResponse)
def html_response():
    return """
    <!DOCTYPE html>
    <html>
        <head><title>FastAPI</title></head>
        <body>
            <h1>Hello from FastAPI!</h1>
        </body>
    </html>
    """

# Текстовый ответ
@app.get('/text', response_class=PlainTextResponse)
def text_response():
    return "This is plain text"

# Перенаправление
@app.get('/redirect')
def redirect():
    return RedirectResponse(url='/json')

# Файловый ответ
@app.get('/download/{filename}')
def download_file(filename: str):
    file_path = Path(f"uploads/{filename}")
    if not file_path.exists():
        raise HTTPException(status_code=404, detail="File not found")
    return FileResponse(
        path=file_path,
        filename=filename,
        media_type='application/octet-stream'
    )

# Потоковый ответ (генератор)
@app.get('/stream')
def stream_response():
    def generate():
        yield "First chunk\n"
        yield "Second chunk\n"
        yield "Third chunk\n"
    return StreamingResponse(generate(), media_type="text/plain")
3.2. Статус-коды
python
from fastapi import status

@app.post('/items', status_code=status.HTTP_201_CREATED)
def create_item():
    return {"id": 1}

@app.delete('/items/{item_id}', status_code=status.HTTP_204_NO_CONTENT)
def delete_item(item_id: int):
    # Удаление элемента
    return  # Тело ответа пустое при 204

@app.put('/items/{item_id}')
def update_item(item_id: int):
    return JSONResponse(
        status_code=status.HTTP_200_OK,
        content={"message": f"Item {item_id} updated"}
    )
3.3. Модели ответа
python
from pydantic import BaseModel
from typing import Optional, List
from enum import Enum

# Вложенные модели
class Address(BaseModel):
    city: str
    street: str
    house: int

class UserProfile(BaseModel):
    id: int
    name: str
    email: str
    address: Optional[Address] = None

# Модель для списка
class UsersList(BaseModel):
    items: List[UserProfile]
    total: int
    page: int

# Enum для статуса
class ItemStatus(str, Enum):
    active = "active"
    inactive = "inactive"
    deleted = "deleted"

class Item(BaseModel):
    id: int
    name: str
    status: ItemStatus

@app.get('/users/{user_id}', response_model=UserProfile)
def get_user(user_id: int):
    return {
        "id": user_id,
        "name": "Иван",
        "email": "ivan@example.com",
        "address": {
            "city": "Москва",
            "street": "Тверская",
            "house": 10
        }
    }

@app.get('/items/{item_id}', response_model=Item)
def get_item(item_id: int):
    return {
        "id": item_id,
        "name": "Ноутбук",
        "status": ItemStatus.active
    }
3.4. Исключение полей из ответа
python
class User(BaseModel):
    id: int
    name: str
    email: str
    password: str
    
    class Config:
        # Исключаем password из JSON
        fields = {
            'password': {'exclude': True}
        }

@app.get('/users/{user_id}', response_model=User)
def get_user(user_id: int):
    # Пароль не попадёт в ответ
    return {
        "id": user_id,
        "name": "Иван",
        "email": "ivan@example.com",
        "password": "secret"
    }

# Альтернативный способ
@app.get('/users/{user_id}/public')
def get_user_public(user_id: int):
    user = {"id": user_id, "name": "Иван", "email": "ivan@example.com", "password": "secret"}
    return {k: v for k, v in user.items() if k != 'password'}
4. Ответы
4.1. Обработка ошибок через HTTPException
python
from fastapi import HTTPException, status

@app.get('/items/{item_id}')
def get_item(item_id: int):
    if item_id < 1:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="ID должен быть больше 0"
        )
    
    item = find_item(item_id)
    if item is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Товар с ID {item_id} не найден"
        )
    
    return item
4.2. Кастомные обработчики ошибок
python
from fastapi import Request
from fastapi.responses import JSONResponse

# Обработчик ValidationError
@app.exception_handler(ValidationError)
async def validation_exception_handler(request: Request, exc: ValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "detail": "Ошибка валидации",
            "errors": exc.errors()
        }
    )

# Обработчик всех исключений
@app.exception_handler(Exception)
async def generic_exception_handler(request: Request, exc: Exception):
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={
            "detail": "Внутренняя ошибка сервера",
            "type": exc.__class__.__name__
        }
    )
4.3. Свои классы исключений
python
class BusinessError(Exception):
    def __init__(self, message: str, code: int = 400):
        self.message = message
        self.code = code

@app.exception_handler(BusinessError)
async def business_error_handler(request: Request, exc: BusinessError):
    return JSONResponse(
        status_code=exc.code,
        content={"detail": exc.message}
    )

@app.get('/withdraw')
def withdraw(amount: float, balance: float = 1000):
    if amount > balance:
        raise BusinessError("Недостаточно средств", 400)
    return {"withdrawn": amount, "remaining": balance - amount}
5. Обработка ошибок
5.1. Простая зависимость
python
from fastapi import Depends

def common_parameters(
    page: int = 1,
    page_size: int = 10,
    sort: str = "asc"
):
    return {
        "page": page,
        "page_size": page_size,
        "sort": sort
    }

@app.get('/items')
def get_items(params: dict = Depends(common_parameters)):
    return params

@app.get('/users')
def get_users(params: dict = Depends(common_parameters)):
    return params
5.2. Зависимость с логикой
python
from fastapi import Depends, HTTPException, status

async def get_current_user(token: str = Header(...)):
    # Имитация проверки токена
    if token != "secret-token":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid token"
        )
    return {"id": 1, "name": "Current User"}

async def get_admin_user(user: dict = Depends(get_current_user)):
    if user['id'] != 1:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Admin access required"
        )
    return user

@app.get('/profile')
def get_profile(user: dict = Depends(get_current_user)):
    return user

@app.get('/admin')
def admin_panel(admin: dict = Depends(get_admin_user)):
    return {"message": "Welcome to admin panel!"}
5.3. Зависимости с yield (для ресурсов)
python
from contextlib import contextmanager

async def get_db():
    db = DatabaseConnection()
    try:
        yield db
    finally:
        db.close()

@app.get('/users')
def get_users(db = Depends(get_db)):
    return db.query("SELECT * FROM users")
6. Зависимости (Dependency Injection)
6.1. Полный пример: менеджер задач
python
from fastapi import FastAPI, HTTPException, Depends, Query, Path, Body, status
from pydantic import BaseModel, Field
from typing import List, Optional
from datetime import datetime
import uuid

app = FastAPI(title="Task Manager API")

# Модели
class TaskCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=100)
    description: Optional[str] = Field(None, max_length=500)
    priority: int = Field(1, ge=1, le=5)

class Task(TaskCreate):
    id: str
    created_at: datetime
    completed: bool = False

class TaskUpdate(BaseModel):
    title: Optional[str] = Field(None, min_length=1, max_length=100)
    description: Optional[str] = None
    priority: Optional[int] = Field(None, ge=1, le=5)
    completed: Optional[bool] = None

# Хранилище в памяти
tasks_db: dict = {}

# Вспомогательные функции
def get_task_or_404(task_id: str) -> Task:
    if task_id not in tasks_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Task {task_id} not found"
        )
    return tasks_db[task_id]

def pagination_params(
    page: int = Query(1, ge=1),
    page_size: int = Query(10, ge=1, le=100)
):
    return {"page": page, "page_size": page_size, "offset": (page - 1) * page_size}

def filter_params(
    completed: Optional[bool] = None,
    priority_min: Optional[int] = Query(None, ge=1, le=5),
    priority_max: Optional[int] = Query(None, ge=1, le=5)
):
    return {"completed": completed, "priority_min": priority_min, "priority_max": priority_max}

# Маршруты
@app.post('/tasks', response_model=Task, status_code=status.HTTP_201_CREATED)
def create_task(task: TaskCreate):
    new_task = Task(
        id=str(uuid.uuid4()),
        created_at=datetime.now(),
        **task.dict()
    )
    tasks_db[new_task.id] = new_task
    return new_task

@app.get('/tasks', response_model=List[Task])
def list_tasks(
    pagination: dict = Depends(pagination_params),
    filters: dict = Depends(filter_params)
):
    tasks = list(tasks_db.values())
    
    # Фильтрация
    if filters['completed'] is not None:
        tasks = [t for t in tasks if t.completed == filters['completed']]
    
    if filters['priority_min'] is not None:
        tasks = [t for t in tasks if t.priority >= filters['priority_min']]
    
    if filters['priority_max'] is not None:
        tasks = [t for t in tasks if t.priority <= filters['priority_max']]
    
    # Пагинация
    offset = pagination['offset']
    limit = pagination['page_size']
    tasks = tasks[offset:offset + limit]
    
    return tasks

@app.get('/tasks/{task_id}', response_model=Task)
def get_task(task: Task = Depends(get_task_or_404)):
    return task

@app.put('/tasks/{task_id}', response_model=Task)
def update_task(
    task_update: TaskUpdate,
    task: Task = Depends(get_task_or_404)
):
    update_data = task_update.dict(exclude_unset=True)
    for field, value in update_data.items():
        setattr(task, field, value)
    tasks_db[task.id] = task
    return task

@app.delete('/tasks/{task_id}', status_code=status.HTTP_204_NO_CONTENT)
def delete_task(task: Task = Depends(get_task_or_404)):
    del tasks_db[task.id]

# Дополнительный эндпоинт
@app.post('/tasks/{task_id}/complete', response_model=Task)
def complete_task(task: Task = Depends(get_task_or_404)):
    task.completed = True
    tasks_db[task.id] = task
    return task
7. Практические примеры
7.1. Пример с полным набором параметров
python
@app.post('/orders')
def create_order(
    # Path parameters
    user_id: int = Path(..., gt=0),
    
    # Query parameters
    priority: int = Query(1, ge=1, le=5),
    
    # Body parameters
    order: dict = Body(...),
    
    # Headers
    authorization: str = Header(...),
    
    # Cookies
    session_id: Optional[str] = Cookie(None),
    
    # Form data
    promo_code: Optional[str] = Form(None)
):
    return {
        "user_id": user_id,
        "priority": priority,
        "order": order,
        "authorization": authorization,
        "session_id": session_id,
        "promo_code": promo_code
    }
8. Контрольные вопросы
Как создать маршрут для GET запроса в FastAPI?

Как получить параметр из пути URL? Приведите пример.

Как получить query-параметры и добавить им валидацию?

Что такое Pydantic и как он используется в FastAPI для валидации тела запроса?

Как получить заголовки запроса в FastAPI?

Как загрузить файл через FastAPI?

Как вернуть HTML страницу вместо JSON?

Как обработать ошибку 404 и вернуть кастомное сообщение?

Что такое dependency injection и как его использовать в FastAPI?

Как создать маршрут, который принимает и JSON, и form-data?

9. Практическое задание
Задание 1 (базовое)
Создайте FastAPI приложение с одним маршрутом /hello, который принимает query-параметр name и возвращает {"message": f"Hello, {name}!"}.

Задание 2 (среднее)
Создайте CRUD API для управления заметками с поддержкой:

Создание заметки (POST /notes) с телом запроса (title, content)

Получение всех заметок (GET /notes) с пагинацией

Получение заметки по ID (GET /notes/{id})

Обновление заметки (PUT /notes/{id})

Удаление заметки (DELETE /notes/{id})

Задание 3 (сложное)
Создайте API для загрузки и скачивания файлов:

POST /upload — загрузка файла, сохранение на диск

GET /files — список всех загруженных файлов

GET /download/{filename} — скачивание файла

DELETE /files/{filename} — удаление файла

10. Шпаргалка
python
# === МАРШРУТЫ ===
@app.get('/path')      # GET
@app.post('/path')     # POST
@app.put('/path')      # PUT
@app.delete('/path')   # DELETE
@app.patch('/path')    # PATCH

# === ПАРАМЕТРЫ ===
# Path
def handler(item_id: int)

# Query
def handler(page: int = 1, q: str = None)

# Body (Pydantic)
def handler(item: Item)

# Headers
def handler(auth: str = Header(...))

# Cookies
def handler(session: str = Cookie(None))

# Files
def handler(file: UploadFile = File(...))

# Forms
def handler(name: str = Form(...))

# === ОТВЕТЫ ===
return dict              # JSON
return HTMLResponse()    # HTML
return PlainTextResponse() # Текст
return FileResponse()    # Файл
return RedirectResponse() # Редирект

# === СТАТУСЫ ===
status_code=201  # Created
status_code=204  # No Content
raise HTTPException(status_code=404, detail="Not found")
Итог лекции
Вы сегодня:

✅ Изучили создание маршрутов и обработчиков в FastAPI

✅ Научились работать с параметрами пути и query-параметрами

✅ Освоили валидацию тела запроса через Pydantic

✅ Узнали о различных типах ответов

✅ Изучили обработку ошибок и dependency injection

✅ Создали полноценный REST API для управления задачами

Теперь вы можете создавать профессиональные API на FastAPI!
