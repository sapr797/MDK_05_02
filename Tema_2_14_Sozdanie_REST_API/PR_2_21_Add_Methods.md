# ПЗ 2.21. Добавление PUT, DELETE методов. Обработка JSON

**Тема:** Веб-фреймворки Python, FastAPI, REST API, полный CRUD, работа с JSON

**Цель работы:**  
Научиться реализовывать полный CRUD функционал на FastAPI (GET, POST, PUT, DELETE), корректно обрабатывать JSON-запросы и ответы, валидировать данные, обрабатывать ошибки.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленный FastAPI и Uvicorn

```bash
pip install fastapi uvicorn
Главная мысль: CRUD — основа любого API. GET для чтения, POST для создания, PUT для обновления, DELETE для удаления. Владея этими четырьмя методами, вы можете построить 90% бэкендов.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. CRUD операции и HTTP методы
Операция	HTTP метод	URL	Тело запроса	Тело ответа
Create (создание)	POST	/resource	Данные нового ресурса	Созданный ресурс
Read (чтение)	GET	/resource	-	Список ресурсов
Read one (чтение одного)	GET	/resource/{id}	-	Один ресурс
Update (обновление)	PUT	/resource/{id}	Полные данные ресурса	Обновлённый ресурс
Partial Update (частичное)	PATCH	/resource/{id}	Часть данных	Обновлённый ресурс
Delete (удаление)	DELETE	/resource/{id}	-	Пустой ответ (204)
1.2. PUT vs PATCH
Метод	Описание	Идемпотентность	Когда использовать
PUT	Заменяет ресурс целиком	✅ Да	Обновление всех полей
PATCH	Частичное обновление	❌ Нет	Обновление отдельных полей
python
# PUT — полная замена
@app.put('/items/{item_id}')
def update_item(item_id: int, item: ItemCreate):
    # Заменяем ВСЕ поля
    items_db[item_id] = Item(id=item_id, **item.dict())
    return items_db[item_id]

# PATCH — частичное обновление
@app.patch('/items/{item_id}')
def patch_item(item_id: int, item: ItemUpdate):
    # Обновляем только переданные поля
    existing = items_db[item_id]
    update_data = item.dict(exclude_unset=True)
    for field, value in update_data.items():
        setattr(existing, field, value)
    return existing
1.3. Статус-коды для CRUD
Операция	Успех	Ошибка клиента	Ошибка сервера
GET	200 OK	404 Not Found	500
POST	201 Created	400 Bad Request, 422	500
PUT	200 OK	400, 404, 422	500
DELETE	204 No Content	404 Not Found	500
1.4. Структура JSON ответов
json
// Успешный ответ (200 OK)
{
    "id": 123,
    "name": "Иван",
    "email": "ivan@example.com"
}

// Ответ с пагинацией
{
    "data": [...],
    "meta": {
        "total": 42,
        "page": 1,
        "limit": 10
    }
}

// Ответ при создании (201 Created)
{
    "id": 123,
    "name": "Новый ресурс"
}

// Ответ при удалении (204 No Content) — пустое тело

// Ошибка валидации (422 Unprocessable Entity)
{
    "detail": [
        {
            "loc": ["body", "email"],
            "msg": "value is not a valid email",
            "type": "value_error.email"
        }
    ]
}
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая полный CRUD на FastAPI.

Техническое задание (нулевой вариант)
Разработайте REST API для управления пользователями с полным CRUD функционалом:

GET /users — список всех пользователей

GET /users/{user_id} — получение пользователя по ID

POST /users — создание нового пользователя

PUT /users/{user_id} — полное обновление пользователя

DELETE /users/{user_id} — удаление пользователя

PATCH /users/{user_id} — частичное обновление (дополнительно)

Эталонная реализация
python
#!/usr/bin/env python3
"""
users_api.py — REST API для управления пользователями (полный CRUD).
FastAPI, GET, POST, PUT, PATCH, DELETE методы, обработка JSON.
"""

from fastapi import FastAPI, HTTPException, Query, Path, status
from pydantic import BaseModel, Field, EmailStr
from typing import List, Optional
from datetime import datetime
import uuid

# ============================================================
# МОДЕЛИ ДАННЫХ (Pydantic)
# ============================================================

class UserCreate(BaseModel):
    """Модель для создания пользователя."""
    name: str = Field(..., min_length=1, max_length=50, description="Полное имя")
    email: EmailStr = Field(..., description="Электронная почта")
    age: int = Field(..., ge=0, le=150, description="Возраст")
    is_active: bool = Field(True, description="Активен ли пользователь")
    
    class Config:
        schema_extra = {
            "example": {
                "name": "Иван Петров",
                "email": "ivan@example.com",
                "age": 25,
                "is_active": True
            }
        }

class UserUpdate(BaseModel):
    """Модель для частичного обновления пользователя."""
    name: Optional[str] = Field(None, min_length=1, max_length=50)
    email: Optional[EmailStr] = None
    age: Optional[int] = Field(None, ge=0, le=150)
    is_active: Optional[bool] = None


class UserResponse(BaseModel):
    """Модель для ответа с данными пользователя."""
    id: str = Field(..., description="Уникальный идентификатор")
    name: str
    email: EmailStr
    age: int
    is_active: bool
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        schema_extra = {
            "example": {
                "id": "123e4567-e89b-12d3-a456-426614174000",
                "name": "Иван Петров",
                "email": "ivan@example.com",
                "age": 25,
                "is_active": True,
                "created_at": "2024-01-15T14:30:25.123456",
                "updated_at": None
            }
        }


class UsersListResponse(BaseModel):
    """Модель для ответа со списком пользователей."""
    data: List[UserResponse]
    total: int
    page: int
    limit: int
    
    @property
    def total_pages(self) -> int:
        return (self.total + self.limit - 1) // self.limit


# ============================================================
# ХРАНИЛИЩЕ ДАННЫХ (в памяти)
# ============================================================

# База данных пользователей
users_db: dict = {}


# ============================================================
# ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ
# ============================================================

def get_user_or_404(user_id: str) -> UserResponse:
    """
    Возвращает пользователя или вызывает HTTPException 404.
    
    Args:
        user_id: ID пользователя
    
    Returns:
        UserResponse: Найденный пользователь
    """
    if user_id not in users_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Пользователь с ID {user_id} не найден"
        )
    return users_db[user_id]


def check_email_unique(email: str, exclude_user_id: Optional[str] = None) -> None:
    """
    Проверяет уникальность email.
    
    Args:
        email: Email для проверки
        exclude_user_id: ID пользователя, который исключается из проверки
    
    Raises:
        HTTPException: 409 Conflict если email уже занят
    """
    for user_id, user in users_db.items():
        if user.email == email:
            if exclude_user_id is None or user_id != exclude_user_id:
                raise HTTPException(
                    status_code=status.HTTP_409_CONFLICT,
                    detail=f"Пользователь с email {email} уже существует"
                )


# ============================================================
# СОЗДАНИЕ ПРИЛОЖЕНИЯ FASTAPI
# ============================================================

app = FastAPI(
    title="Users API",
    description="REST API для управления пользователями (полный CRUD)",
    version="2.0.0",
    docs_url="/api/docs",
    redoc_url="/api/redoc"
)


# ============================================================
# GET ЗАПРОСЫ (чтение)
# ============================================================

@app.get(
    '/users',
    response_model=UsersListResponse,
    summary="Получить список пользователей",
    description="Возвращает список всех пользователей с пагинацией и фильтрацией"
)
def get_users(
    page: int = Query(1, ge=1, description="Номер страницы"),
    limit: int = Query(10, ge=1, le=100, description="Количество на странице"),
    is_active: Optional[bool] = Query(None, description="Фильтр по статусу"),
    search: Optional[str] = Query(None, min_length=1, description="Поиск по имени")
):
    """
    Получение списка пользователей с пагинацией и фильтрацией.
    
    Args:
        page: Номер страницы (начиная с 1)
        limit: Количество записей на странице
        is_active: Фильтр по статусу активности
        search: Поиск по имени (частичное совпадение)
    
    Returns:
        UsersListResponse: Список пользователей и метаданные
    """
    # Фильтрация
    users = list(users_db.values())
    
    if is_active is not None:
        users = [u for u in users if u.is_active == is_active]
    
    if search:
        search_lower = search.lower()
        users = [u for u in users if search_lower in u.name.lower()]
    
    # Сортировка по дате создания (новые сверху)
    users.sort(key=lambda u: u.created_at, reverse=True)
    
    # Пагинация
    total = len(users)
    offset = (page - 1) * limit
    paginated_users = users[offset:offset + limit]
    
    return UsersListResponse(
        data=paginated_users,
        total=total,
        page=page,
        limit=limit
    )


@app.get(
    '/users/{user_id}',
    response_model=UserResponse,
    summary="Получить пользователя по ID",
    description="Возвращает полную информацию о пользователе"
)
def get_user(
    user_id: str = Path(..., description="ID пользователя")
):
    """
    Получение пользователя по ID.
    
    Args:
        user_id: ID пользователя
    
    Returns:
        UserResponse: Данные пользователя
    """
    return get_user_or_404(user_id)


# ============================================================
# POST ЗАПРОС (создание)
# ============================================================

@app.post(
    '/users',
    response_model=UserResponse,
    status_code=status.HTTP_201_CREATED,
    summary="Создать пользователя",
    description="Создаёт нового пользователя и возвращает его данные"
)
def create_user(user_data: UserCreate):
    """
    Создание нового пользователя.
    
    Args:
        user_data: Данные для создания пользователя
    
    Returns:
        UserResponse: Созданный пользователь
    """
    # Проверка уникальности email
    check_email_unique(user_data.email)
    
    # Создание пользователя
    new_user = UserResponse(
        id=str(uuid.uuid4()),
        created_at=datetime.now(),
        **user_data.dict()
    )
    
    # Сохранение в "базу данных"
    users_db[new_user.id] = new_user
    
    return new_user


# ============================================================
# PUT ЗАПРОС (полное обновление)
# ============================================================

@app.put(
    '/users/{user_id}',
    response_model=UserResponse,
    summary="Полностью обновить пользователя",
    description="Заменяет все данные пользователя"
)
def update_user(
    user_data: UserCreate,
    user_id: str = Path(..., description="ID пользователя")
):
    """
    Полное обновление пользователя (PUT).
    
    Args:
        user_id: ID пользователя
        user_data: Новые данные пользователя
    
    Returns:
        UserResponse: Обновлённый пользователь
    """
    # Проверка существования
    existing_user = get_user_or_404(user_id)
    
    # Проверка уникальности email (исключая текущего пользователя)
    check_email_unique(user_data.email, exclude_user_id=user_id)
    
    # Обновление пользователя
    updated_user = UserResponse(
        id=user_id,
        created_at=existing_user.created_at,
        updated_at=datetime.now(),
        **user_data.dict()
    )
    
    # Сохранение
    users_db[user_id] = updated_user
    
    return updated_user


# ============================================================
# PATCH ЗАПРОС (частичное обновление)
# ============================================================

@app.patch(
    '/users/{user_id}',
    response_model=UserResponse,
    summary="Частично обновить пользователя",
    description="Обновляет только переданные поля пользователя"
)
def patch_user(
    user_data: UserUpdate,
    user_id: str = Path(..., description="ID пользователя")
):
    """
    Частичное обновление пользователя (PATCH).
    
    Args:
        user_id: ID пользователя
        user_data: Данные для обновления (только переданные поля)
    
    Returns:
        UserResponse: Обновлённый пользователь
    """
    # Проверка существования
    existing_user = get_user_or_404(user_id)
    
    # Проверка уникальности email (если он передан)
    if user_data.email is not None:
        check_email_unique(user_data.email, exclude_user_id=user_id)
    
    # Обновление только переданных полей
    update_data = user_data.dict(exclude_unset=True)
    
    updated_user = UserResponse(
        id=existing_user.id,
        name=update_data.get('name', existing_user.name),
        email=update_data.get('email', existing_user.email),
        age=update_data.get('age', existing_user.age),
        is_active=update_data.get('is_active', existing_user.is_active),
        created_at=existing_user.created_at,
        updated_at=datetime.now()
    )
    
    # Сохранение
    users_db[user_id] = updated_user
    
    return updated_user


# ============================================================
# DELETE ЗАПРОС (удаление)
# ============================================================

@app.delete(
    '/users/{user_id}',
    status_code=status.HTTP_204_NO_CONTENT,
    summary="Удалить пользователя",
    description="Удаляет пользователя из системы"
)
def delete_user(
    user_id: str = Path(..., description="ID пользователя")
):
    """
    Удаление пользователя.
    
    Args:
        user_id: ID пользователя
    
    Returns:
        None (пустой ответ)
    """
    # Проверка существования
    get_user_or_404(user_id)
    
    # Удаление
    del users_db[user_id]


# ============================================================
# ДОПОЛНИТЕЛЬНО: ЭНДПОИНТЫ ДЛЯ СТАТИСТИКИ
# ============================================================

@app.get(
    '/stats',
    summary="Статистика пользователей",
    description="Возвращает статистику по пользователям"
)
def get_stats():
    """Возвращает статистику по пользователям."""
    total = len(users_db)
    active = sum(1 for u in users_db.values() if u.is_active)
    
    ages = [u.age for u in users_db.values()]
    avg_age = sum(ages) / len(ages) if ages else 0
    
    return {
        "total_users": total,
        "active_users": active,
        "inactive_users": total - active,
        "average_age": round(avg_age, 1),
        "users_by_age": {
            "under_18": sum(1 for u in users_db.values() if u.age < 18),
            "18_30": sum(1 for u in users_db.values() if 18 <= u.age <= 30),
            "31_50": sum(1 for u in users_db.values() if 31 <= u.age <= 50),
            "over_50": sum(1 for u in users_db.values() if u.age > 50)
        }
    }


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
Тест 1: POST — создание пользователя
bash
curl -X POST http://localhost:8000/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Иван Петров",
    "email": "ivan@example.com",
    "age": 25,
    "is_active": true
  }'
Ответ:

json
{
    "id": "abc123...",
    "name": "Иван Петров",
    "email": "ivan@example.com",
    "age": 25,
    "is_active": true,
    "created_at": "2024-01-15T14:30:25.123456",
    "updated_at": null
}
Тест 2: GET — список всех пользователей
bash
curl -X GET http://localhost:8000/users
Ответ:

json
{
    "data": [
        {
            "id": "abc123...",
            "name": "Иван Петров",
            "email": "ivan@example.com",
            "age": 25,
            "is_active": true,
            "created_at": "2024-01-15T14:30:25.123456",
            "updated_at": null
        }
    ],
    "total": 1,
    "page": 1,
    "limit": 10
}
Тест 3: GET — конкретный пользователь
bash
curl -X GET http://localhost:8000/users/abc123...
Тест 4: PUT — полное обновление
bash
curl -X PUT http://localhost:8000/users/abc123... \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Иван Сергеевич Петров",
    "email": "ivan.petrov@example.com",
    "age": 26,
    "is_active": false
  }'
Тест 5: PATCH — частичное обновление
bash
curl -X PATCH http://localhost:8000/users/abc123... \
  -H "Content-Type: application/json" \
  -d '{
    "name": "И.С. Петров"
  }'
Тест 6: DELETE — удаление пользователя
bash
curl -X DELETE http://localhost:8000/users/abc123...
Ответ: (пустой, статус 204)

3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (GET, POST, PUT, DELETE)

Варианты 9-17: средний (+ PATCH, фильтрация, пагинация)

Варианты 18-25: сложный (+ поиск, статистика, связи)

Варианты 1-8 (Базовый уровень)
№	Ресурс	Поля модели	Дополнительно
1	Продукты	name, price, category	PUT, DELETE
2	Задачи	title, description, status	PUT, DELETE
3	Книги	title, author, year	PUT, DELETE
4	Студенты	name, email, course	PUT, DELETE
5	Сотрудники	name, position, salary	PUT, DELETE
6	Фильмы	title, director, year, rating	PUT, DELETE
7	Рестораны	name, cuisine, rating	PUT, DELETE
8	Автомобили	brand, model, year, price	PUT, DELETE
Варианты 9-17 (Средний уровень)
№	Ресурс	Дополнительные эндпоинты	Особенности
9	Заметки	PATCH, поиск по тегам	+ пагинация
10	Клиенты	PATCH, фильтрация по статусу	+ пагинация
11	Заказы	PATCH, фильтрация по дате	+ сортировка
12	Комментарии	PATCH, фильтрация по автору	+ пагинация
13	События	PATCH, фильтрация по дате	+ пагинация
14	Курсы	PATCH, фильтрация по уровню	+ пагинация
15	Отели	PATCH, фильтрация по звёздам	+ пагинация
16	Транзакции	PATCH, фильтрация по сумме	+ пагинация
17	Артисты	PATCH, фильтрация по жанру	+ пагинация
Варианты 18-25 (Сложный уровень)
№	Ресурс	Дополнительные требования
18	Пользователи + роли	Связи между ресурсами
19	Посты + комментарии	Вложенные ресурсы
20	Товары + категории	Связь многие-ко-многим
21	Заказы + товары	Корзина, связи
22	Библиотека + книги	Связь один-ко-многим
23	Курсы + студенты	Зачисление, отчисление
24	Проекты + задачи	Иерархия задач
25	Компании + сотрудники	Организационная структура
Пример варианта 18 (связи):

python
# Модели
class Role(BaseModel):
    id: int
    name: str
    permissions: List[str]

class UserWithRole(UserResponse):
    role: Optional[Role] = None

# Дополнительные эндпоинты
@app.post('/users/{user_id}/role')
def assign_role(user_id: str, role_id: int):
    """Назначить роль пользователю."""
    pass

@app.get('/users/by-role/{role_id}')
def get_users_by_role(role_id: int):
    """Получить всех пользователей с ролью."""
    pass
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	API не реализует все CRUD операции
3 (удовлетворительно)	Реализованы GET, POST, PUT, DELETE
4 (хорошо)	+ PATCH, базовая фильтрация, пагинация
5 (отлично)	Полный CRUD + поиск, фильтрация, пагинация, обработка ошибок, связи
5. Шпаргалка
5.1. Шаблон полного CRUD
python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List, Optional
from datetime import datetime

app = FastAPI()

# Модели
class ItemCreate(BaseModel):
    name: str
    price: float

class Item(ItemCreate):
    id: int
    created_at: datetime

# Хранилище
items_db = {}
counter = 0

# GET all
@app.get('/items', response_model=List[Item])
def get_items():
    return list(items_db.values())

# GET one
@app.get('/items/{item_id}', response_model=Item)
def get_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(404, "Not found")
    return items_db[item_id]

# POST
@app.post('/items', response_model=Item, status_code=201)
def create_item(item: ItemCreate):
    global counter
    counter += 1
    new_item = Item(id=counter, created_at=datetime.now(), **item.dict())
    items_db[counter] = new_item
    return new_item

# PUT
@app.put('/items/{item_id}', response_model=Item)
def update_item(item_id: int, item: ItemCreate):
    if item_id not in items_db:
        raise HTTPException(404, "Not found")
    updated_item = Item(id=item_id, created_at=items_db[item_id].created_at, **item.dict())
    items_db[item_id] = updated_item
    return updated_item

# DELETE
@app.delete('/items/{item_id}', status_code=204)
def delete_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(404, "Not found")
    del items_db[item_id]
5.2. Тестирование CRUD
bash
# Создание
POST /items -d '{"name": "Ноутбук", "price": 50000}'

# Чтение всех
GET /items

# Чтение одного
GET /items/1

# Обновление
PUT /items/1 -d '{"name": "Игровой ноутбук", "price": 70000}'

# Удаление
DELETE /items/1
Карточка студента
text
ПЗ 2.21. ДОБАВЛЕНИЕ PUT, DELETE МЕТОДОВ. ОБРАБОТКА JSON

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Ресурс: _________________________________

=== CRUD ОПЕРАЦИИ ===

□ GET /resource (список)
□ GET /resource/{id} (один)
□ POST /resource (создание)
□ PUT /resource/{id} (полное обновление)
□ PATCH /resource/{id} (частичное обновление)
□ DELETE /resource/{id} (удаление)

=== ДОПОЛНИТЕЛЬНЫЕ ВОЗМОЖНОСТИ ===

□ Пагинация (page, limit)
□ Фильтрация (query параметры)
□ Поиск (search)
□ Сортировка (sort)
□ Валидация данных (Pydantic)
□ Обработка ошибок (404, 409, 422)

=== ОТЧЁТ ===

Ссылка на репозиторий: _____________
Скриншоты тестирования: _____________

Дата выполнения: _____________
