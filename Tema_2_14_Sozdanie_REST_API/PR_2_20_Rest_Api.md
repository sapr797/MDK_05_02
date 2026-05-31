# ПЗ 2.20. Создание простого REST API на FastAPI (GET, POST)

**Тема:** Веб-фреймворки Python, FastAPI, REST API, маршруты, обработчики

**Цель работы:**  
Научиться создавать простые REST API на FastAPI, реализовывать обработчики GET и POST запросов, работать с параметрами пути, query-параметрами и телом запроса, возвращать JSON-ответы.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленный FastAPI и Uvicorn

```bash
pip install fastapi uvicorn
Главная мысль: FastAPI превращает аннотации типов в мощный инструмент валидации и документации. Начните с малого — GET и POST запросов достаточно для большинства задач.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Что такое FastAPI
FastAPI — современный веб-фреймворк для создания API на Python.

Основные преимущества:

Преимущество	Описание
Высокая производительность	Один из самых быстрых Python-фреймворков
Автоматическая документация	Swagger UI и ReDoc генерируются автоматически
Встроенная валидация	На основе Pydantic и аннотаций типов
Асинхронность	Поддержка async/await
Автодополнение	Работает с аннотациями типов
1.2. GET и POST запросы
Метод	Назначение	Где находятся данные	Идемпотентность
GET	Получение данных	URL (path, query)	✅ Да
POST	Создание данных	Тело запроса (body)	❌ Нет
1.3. Структура URL
text
https://api.example.com/users/123?page=2&limit=10
\_________________/ \____/ \ / \_________________/
      База        │Ресурс│ID │   Query параметры
                  └──────┴───┘
1.4. Запуск FastAPI приложения
bash
# Команда для запуска
uvicorn main:app --reload

# --reload — автоматический перезапуск при изменениях
# main — имя файла (main.py)
# app — переменная FastAPI в файле
1.5. Структура проекта
text
my_api/
├── main.py              # Главный файл приложения
├── requirements.txt     # Зависимости
└── venv/               # Виртуальное окружение
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая создание простого REST API на FastAPI.

Техническое задание (нулевой вариант)
Разработайте REST API для управления списком задач (To-Do list). API должно поддерживать:

GET /tasks — получение списка всех задач

GET /tasks/{task_id} — получение задачи по ID

POST /tasks — создание новой задачи

GET /tasks/search — поиск задач по названию (query-параметр q)

Эталонная реализация
python
#!/usr/bin/env python3
"""
todo_api.py — REST API для управления списком задач (To-Do list).
FastAPI, GET и POST запросы.
"""

from fastapi import FastAPI, HTTPException, Query, Path, status
from pydantic import BaseModel, Field
from typing import List, Optional
from datetime import datetime
import uuid

# ============================================================
# МОДЕЛИ ДАННЫХ (Pydantic)
# ============================================================

class TaskCreate(BaseModel):
    """Модель для создания задачи."""
    title: str = Field(..., min_length=1, max_length=100, description="Название задачи")
    description: Optional[str] = Field(None, max_length=500, description="Описание задачи")
    priority: int = Field(1, ge=1, le=5, description="Приоритет (1-5)")

class Task(TaskCreate):
    """Модель задачи с автоматическими полями."""
    id: str = Field(..., description="Уникальный идентификатор")
    created_at: datetime = Field(..., description="Дата создания")
    completed: bool = Field(False, description="Статус выполнения")

class TaskUpdate(BaseModel):
    """Модель для обновления задачи."""
    title: Optional[str] = Field(None, min_length=1, max_length=100)
    description: Optional[str] = None
    priority: Optional[int] = Field(None, ge=1, le=5)
    completed: Optional[bool] = None


# ============================================================
# ХРАНИЛИЩЕ ДАННЫХ (в памяти)
# ============================================================

# База данных задач (имитация)
tasks_db: dict = {}

# Счётчик для ID (только для демонстрации, в реальности лучше UUID)
counter = 0


# ============================================================
# ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ
# ============================================================

def get_task_or_404(task_id: str) -> Task:
    """
    Возвращает задачу или вызывает HTTPException 404.
    
    Args:
        task_id: ID задачи
    
    Returns:
        Task: Найденная задача
    
    Raises:
        HTTPException: 404 если задача не найдена
    """
    if task_id not in tasks_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Задача с ID {task_id} не найдена"
        )
    return tasks_db[task_id]


# ============================================================
# СОЗДАНИЕ ПРИЛОЖЕНИЯ FASTAPI
# ============================================================

app = FastAPI(
    title="To-Do List API",
    description="Простейшее REST API для управления задачами",
    version="1.0.0",
    docs_url="/api/docs",
    redoc_url="/api/redoc"
)


# ============================================================
# GET ЗАПРОСЫ (получение данных)
# ============================================================

@app.get(
    '/',
    summary="Корневой эндпоинт",
    description="Возвращает приветственное сообщение и ссылки на документацию"
)
def root():
    """Корневой эндпоинт API."""
    return {
        "message": "Добро пожаловать в To-Do List API!",
        "docs": "/api/docs",
        "endpoints": {
            "GET /tasks": "Получить все задачи",
            "GET /tasks/{id}": "Получить задачу по ID",
            "POST /tasks": "Создать новую задачу",
            "GET /tasks/search": "Поиск задач по названию"
        }
    }


@app.get(
    '/tasks',
    response_model=List[Task],
    summary="Получить все задачи",
    description="Возвращает список всех задач в системе"
)
def get_all_tasks():
    """
    Получение списка всех задач.
    
    Returns:
        List[Task]: Список задач
    """
    return list(tasks_db.values())


@app.get(
    '/tasks/{task_id}',
    response_model=Task,
    summary="Получить задачу по ID",
    description="Возвращает задачу с указанным идентификатором"
)
def get_task_by_id(
    task_id: str = Path(..., description="ID задачи")
):
    """
    Получение задачи по ID.
    
    Args:
        task_id: ID задачи
    
    Returns:
        Task: Найденная задача
    """
    return get_task_or_404(task_id)


@app.get(
    '/tasks/search',
    response_model=List[Task],
    summary="Поиск задач",
    description="Поиск задач по названию (частичное совпадение)"
)
def search_tasks(
    q: str = Query(..., min_length=1, description="Поисковый запрос"),
    priority: Optional[int] = Query(None, ge=1, le=5, description="Фильтр по приоритету")
):
    """
    Поиск задач по названию.
    
    Args:
        q: Поисковый запрос
        priority: Фильтр по приоритету (опционально)
    
    Returns:
        List[Task]: Список задач, удовлетворяющих поиску
    """
    results = []
    
    for task in tasks_db.values():
        # Поиск по названию (регистронезависимый)
        if q.lower() in task.title.lower():
            # Фильтрация по приоритету
            if priority is None or task.priority == priority:
                results.append(task)
    
    return results


# ============================================================
# POST ЗАПРОСЫ (создание данных)
# ============================================================

@app.post(
    '/tasks',
    response_model=Task,
    status_code=status.HTTP_201_CREATED,
    summary="Создать задачу",
    description="Создаёт новую задачу и возвращает её"
)
def create_task(task_data: TaskCreate):
    """
    Создание новой задачи.
    
    Args:
        task_data: Данные для создания задачи
    
    Returns:
        Task: Созданная задача
    """
    global counter
    
    # Генерация ID
    counter += 1
    task_id = str(counter)
    
    # Создание задачи
    new_task = Task(
        id=task_id,
        created_at=datetime.now(),
        **task_data.dict()
    )
    
    # Сохранение в "базу данных"
    tasks_db[task_id] = new_task
    
    return new_task


# ============================================================
# ДОПОЛНИТЕЛЬНО: ОБРАБОТЧИКИ ОШИБОК
# ============================================================

@app.exception_handler(HTTPException)
async def http_exception_handler(request, exc):
    """Кастомный обработчик HTTP исключений."""
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": True,
            "code": exc.status_code,
            "message": exc.detail
        }
    )


# ============================================================
# ЗАПУСК (для отладки)
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
Тест 1: GET запрос к корневому эндпоинту
bash
curl -X GET http://localhost:8000/
Ответ:

json
{
    "message": "Добро пожаловать в To-Do List API!",
    "docs": "/api/docs",
    "endpoints": {
        "GET /tasks": "Получить все задачи",
        "GET /tasks/{id}": "Получить задачу по ID",
        "POST /tasks": "Создать новую задачу",
        "GET /tasks/search": "Поиск задач по названию"
    }
}
Тест 2: POST запрос (создание задачи)
bash
curl -X POST http://localhost:8000/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Изучить FastAPI",
    "description": "Понять основы маршрутизации и валидации",
    "priority": 5
  }'
Ответ:

json
{
    "id": "1",
    "title": "Изучить FastAPI",
    "description": "Понять основы маршрутизации и валидации",
    "priority": 5,
    "created_at": "2024-01-15T14:30:25.123456",
    "completed": false
}
Тест 3: Создание нескольких задач
bash
curl -X POST http://localhost:8000/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Написать код", "priority": 4}'

curl -X POST http://localhost:8000/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Запустить тесты", "description": "Проверить все эндпоинты", "priority": 3}'
Тест 4: GET запрос (все задачи)
bash
curl -X GET http://localhost:8000/tasks
Ответ:

json
[
    {
        "id": "1",
        "title": "Изучить FastAPI",
        "description": "Понять основы маршрутизации и валидации",
        "priority": 5,
        "created_at": "2024-01-15T14:30:25.123456",
        "completed": false
    },
    {
        "id": "2",
        "title": "Написать код",
        "description": null,
        "priority": 4,
        "created_at": "2024-01-15T14:31:10.654321",
        "completed": false
    },
    {
        "id": "3",
        "title": "Запустить тесты",
        "description": "Проверить все эндпоинты",
        "priority": 3,
        "created_at": "2024-01-15T14:31:45.987654",
        "completed": false
    }
]
Тест 5: GET запрос (задача по ID)
bash
curl -X GET http://localhost:8000/tasks/2
Ответ:

json
{
    "id": "2",
    "title": "Написать код",
    "description": null,
    "priority": 4,
    "created_at": "2024-01-15T14:31:10.654321",
    "completed": false
}
Тест 6: GET запрос (поиск)
bash
curl -X GET "http://localhost:8000/tasks/search?q=fastapi"
Ответ:

json
[
    {
        "id": "1",
        "title": "Изучить FastAPI",
        "description": "Понять основы маршрутизации и валидации",
        "priority": 5,
        "created_at": "2024-01-15T14:30:25.123456",
        "completed": false
    }
]
Тест 7: GET запрос (поиск с фильтром)
bash
curl -X GET "http://localhost:8000/tasks/search?q=а&priority=4"
Ответ:

json
[
    {
        "id": "2",
        "title": "Написать код",
        "description": null,
        "priority": 4,
        "created_at": "2024-01-15T14:31:10.654321",
        "completed": false
    }
]
Автоматическая документация
После запуска приложения документация доступна по адресам:

Swagger UI: http://localhost:8000/api/docs

ReDoc: http://localhost:8000/api/redoc

3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (GET, POST для одного ресурса)

Варианты 9-17: средний (GET, POST + поиск, фильтрация)

Варианты 18-25: сложный (GET, POST + валидация, несколько ресурсов)

Варианты 1-8 (Базовый уровень)
№	Тема API	GET эндпоинты	POST эндпоинты	Поля модели
1	Книги	/books, /books/{id}	/books	title, author, year
2	Фильмы	/movies, /movies/{id}	/movies	title, director, year, rating
3	Пользователи	/users, /users/{id}	/users	name, email, age
4	Товары	/products, /products/{id}	/products	name, price, category
5	Студенты	/students, /students/{id}	/students	name, age, course
6	Сотрудники	/employees, /employees/{id}	/employees	name, position, salary
7	Рецепты	/recipes, /recipes/{id}	/recipes	title, cuisine, prep_time
8	Города	/cities, /cities/{id}	/cities	name, country, population
Пример варианта 1:

python
# Модели
class BookCreate(BaseModel):
    title: str = Field(..., min_length=1)
    author: str = Field(..., min_length=1)
    year: int = Field(..., ge=1000, le=2025)

class Book(BookCreate):
    id: int
    created_at: datetime

# Эндпоинты
@app.get('/books', response_model=List[Book])
def get_books(): ...

@app.get('/books/{book_id}', response_model=Book)
def get_book(book_id: int): ...

@app.post('/books', response_model=Book, status_code=201)
def create_book(book: BookCreate): ...
Варианты 9-17 (Средний уровень)
№	Тема API	Дополнительные эндпоинты	Особенности
9	Заметки	/notes/search?q=	Поиск по содержимому
10	Задачи	/tasks?status=	Фильтрация по статусу
11	Игры	/games?platform=	Фильтрация по платформе
12	Песни	/songs?artist=	Фильтрация по исполнителю
13	Рестораны	/restaurants?rating_min=	Фильтрация по рейтингу
14	Курсы	/courses?level=	Фильтрация по уровню
15	Автомобили	/cars?year_from=	Фильтрация по году
16	Отели	/hotels?stars_min=	Фильтрация по звёздам
17	События	/events?date_from=	Фильтрация по дате
Пример варианта 9:

python
@app.get('/notes/search')
def search_notes(
    q: str = Query(..., min_length=1),
    tag: Optional[str] = None
):
    """Поиск заметок по тексту и тегу."""
    results = []
    for note in notes_db.values():
        if q.lower() in note.content.lower():
            if tag is None or tag in note.tags:
                results.append(note)
    return results
Варианты 18-25 (Сложный уровень)
№	Тема API	Дополнительные требования
18	Блог	GET /posts, /posts/{id}, /posts/search; POST /posts + теги
19	Интернет-магазин	GET /products, /products/{id}, /products/category/{cat}; POST /products
20	Библиотека	GET /books, /books/{id}, /books/author/{name}; POST /books
21	Афиша	GET /events, /events/{id}, /events/venue/{name}; POST /events
22	HR-система	GET /employees, /employees/{id}, /employees/department/{dept}; POST /employees
23	CRM	GET /customers, /customers/{id}, /customers/search; POST /customers
24	Образование	GET /courses, /courses/{id}, /courses/instructor/{name}; POST /courses
25	Медицина	GET /doctors, /doctors/{id}, /doctors/specialty/{spec}; POST /doctors
Пример варианта 18:

python
class PostCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    content: str = Field(..., min_length=1)
    tags: List[str] = Field(default_factory=list)

class Post(PostCreate):
    id: int
    author: str
    created_at: datetime
    views: int = 0
    likes: int = 0

# Дополнительные эндпоинты
@app.post('/posts/{post_id}/like')
def like_post(post_id: int):
    """Увеличить счётчик лайков."""
    post = get_post_or_404(post_id)
    post.likes += 1
    return {"likes": post.likes}

@app.get('/posts/popular')
def get_popular_posts(limit: int = 5):
    """Получить самые популярные посты (по просмотрам)."""
    posts = sorted(tasks_db.values(), key=lambda p: p.views, reverse=True)
    return posts[:limit]
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	API не работает или не обрабатывает запросы
3 (удовлетворительно)	Работают GET и POST запросы для одного ресурса
4 (хорошо)	Работают GET, POST + поиск или фильтрация
5 (отлично)	Полный функционал, валидация, обработка ошибок, документация
5. Шпаргалка
5.1. Базовая структура FastAPI приложения
python
from fastapi import FastAPI, HTTPException, Query, Path
from pydantic import BaseModel
from typing import List, Optional

app = FastAPI()

# Модель для создания
class ItemCreate(BaseModel):
    name: str
    price: float

# Модель для ответа
class Item(ItemCreate):
    id: int

# Хранилище
items_db = {}

# GET все
@app.get('/items', response_model=List[Item])
def get_items():
    return list(items_db.values())

# GET один
@app.get('/items/{item_id}', response_model=Item)
def get_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Not found")
    return items_db[item_id]

# POST
@app.post('/items', response_model=Item, status_code=201)
def create_item(item: ItemCreate):
    new_id = len(items_db) + 1
    new_item = Item(id=new_id, **item.dict())
    items_db[new_id] = new_item
    return new_item
5.2. Запуск приложения
bash
# В терминале
uvicorn main:app --reload

# Или из Python
if __name__ == "__main__":
    import uvicorn
    uvicorn.run("main:app", host="127.0.0.1", port=8000, reload=True)
5.3. Тестирование с curl
bash
# GET
curl -X GET http://localhost:8000/items

# GET с параметром
curl -X GET "http://localhost:8000/items/search?q=book"

# POST
curl -X POST http://localhost:8000/items \
  -H "Content-Type: application/json" \
  -d '{"name": "Ноутбук", "price": 50000}'
5.4. Параметры FastAPI
python
@app.get('/example')
def example(
    # Query параметры
    page: int = Query(1, ge=1),
    q: Optional[str] = Query(None, min_length=1),
    
    # Path параметры
    item_id: int = Path(..., gt=0)
):
    pass
Карточка студента
text
ПЗ 2.20. СОЗДАНИЕ ПРОСТОГО REST API НА FASTAPI (GET, POST)

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Тема API: _________________________________

=== GET ЭНДПОИНТЫ ===

□ / (корневой)
□ /resource (список)
□ /resource/{id} (один)
□ /resource/search (поиск)

=== POST ЭНДПОИНТ ===

□ /resource (создание)

=== МОДЕЛИ ===

Модель для создания: _____________
Модель для ответа: _______________

=== ПРОВЕРКА ===

□ curl GET запросы работают
□ curl POST запрос работает
□ Ошибка 404 при неверном ID
□ Валидация данных (Pydantic)
□ Автоматическая документация (Swagger)

=== ОТЧЁТ ===

Ссылка на репозиторий: _____________
Скриншоты запросов: _____________

Дата выполнения: _____________
