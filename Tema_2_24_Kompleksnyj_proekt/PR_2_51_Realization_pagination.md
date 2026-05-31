# ПЗ 2.51. Реализация пагинации и фильтрации в REST API

**Тема:** REST API, пагинация, фильтрация, сортировка, производительность

**Цель работы:**  
Научиться реализовывать пагинацию, фильтрацию и сортировку в REST API, использовать различные стратегии пагинации, оптимизировать запросы к базе данных.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `fastapi`, `uvicorn`, `sqlalchemy`, `pydantic`

```bash
pip install fastapi uvicorn sqlalchemy pydantic
Главная мысль: API без пагинации — это бомба замедленного действия. При росте данных он неизбежно упадет под тяжестью ответа.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Стратегии пагинации
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    СТРАТЕГИИ ПАГИНАЦИИ                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. OFFSET/LIMIT (страничная)                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  GET /items?page=2&limit=10                                         │   │
│  │  SELECT * FROM items LIMIT 10 OFFSET 10                             │   │
│  │  ➕ Простота реализации                                              │   │
│  │  ➖ Проблемы с производительностью при больших OFFSET                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  2. KEYSET (курсорная)                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  GET /items?cursor=2024-01-15&limit=10                              │   │
│  │  SELECT * FROM items WHERE created_at < '2024-01-15' LIMIT 10       │   │
│  │  ➕ Высокая производительность при больших данных                    │   │
│  │  ➖ Сложность реализации, ограничения сортировки                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  3. СЕКВЕНЦИАЛЬНАЯ (по ID)                                                 │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  GET /items?since=100&limit=10                                      │   │
│  │  SELECT * FROM items WHERE id > 100 LIMIT 10                        │   │
│  │  ➕ Производительность, простота                                     │   │
│  │  ➖ Привязка к ID, нет пропусков                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
1.2. Типы фильтрации
Тип	Описание	Пример
Точное совпадение	Равенство значению	status=active
Диапазон	Между значениями	price_min=100&price_max=1000
Поиск	Частичное совпадение	search=laptop
Множественный выбор	Несколько значений	category=1,2,3
Дата/время	До/после даты	created_after=2024-01-01
Булевы	Истина/ложь	is_active=true
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая реализацию пагинации и фильтрации.

Техническое задание (нулевой вариант)
Разработайте API для управления товарами с поддержкой:

Пагинации (страничная и курсорная)

Фильтрации по категории, цене, наличию

Поиска по названию и описанию

Сортировки по цене, дате добавления, популярности

Возврата метаинформации (всего записей, страниц)

Эталонная реализация
python
#!/usr/bin/env python3
"""
products_api.py — API с пагинацией и фильтрацией.
"""

from fastapi import FastAPI, Query, HTTPException, status
from pydantic import BaseModel, Field
from sqlalchemy import create_engine, Column, Integer, String, Float, Boolean, DateTime, func
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from datetime import datetime
from typing import Optional, List, Dict, Any
from enum import Enum
import math

# ============================================================
# НАСТРОЙКА БАЗЫ ДАННЫХ
# ============================================================

DATABASE_URL = "sqlite:///./products.db"
engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()


# ============================================================
# МОДЕЛЬ БАЗЫ ДАННЫХ
# ============================================================

class ProductDB(Base):
    __tablename__ = "products"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(200), nullable=False, index=True)
    description = Column(String(1000))
    price = Column(Float, nullable=False)
    category = Column(String(50), nullable=False, index=True)
    stock = Column(Integer, default=0)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    views = Column(Integer, default=0)


# Создание таблиц
Base.metadata.create_all(bind=engine)


# ============================================================
# PYDANTIC СХЕМЫ
# ============================================================

class ProductResponse(BaseModel):
    id: int
    name: str
    description: Optional[str]
    price: float
    category: str
    stock: int
    is_active: bool
    created_at: datetime
    views: int
    
    class Config:
        from_attributes = True


class ProductListResponse(BaseModel):
    data: List[ProductResponse]
    total: int
    page: int
    limit: int
    pages: int
    has_next: bool
    has_prev: bool


class CursorResponse(BaseModel):
    data: List[ProductResponse]
    next_cursor: Optional[str]
    has_more: bool
    limit: int


# ============================================================
# КЛАССЫ ДЛЯ ПАГИНАЦИИ
# ============================================================

class SortField(str, Enum):
    """Поля для сортировки."""
    ID = "id"
    PRICE = "price"
    CREATED_AT = "created_at"
    VIEWS = "views"


class SortOrder(str, Enum):
    """Порядок сортировки."""
    ASC = "asc"
    DESC = "desc"


class PaginationParams:
    """Параметры страничной пагинации."""
    
    def __init__(
        self,
        page: int = Query(1, ge=1, description="Номер страницы"),
        limit: int = Query(20, ge=1, le=100, description="Записей на странице"),
        sort_by: SortField = Query(SortField.ID, description="Поле сортировки"),
        sort_order: SortOrder = Query(SortOrder.ASC, description="Порядок сортировки")
    ):
        self.page = page
        self.limit = limit
        self.sort_by = sort_by
        self.sort_order = sort_order
        self.offset = (page - 1) * limit


class CursorParams:
    """Параметры курсорной пагинации."""
    
    def __init__(
        self,
        cursor: Optional[str] = Query(None, description="Курсор (закодированное значение)"),
        limit: int = Query(20, ge=1, le=100, description="Записей на странице"),
        sort_by: SortField = Query(SortField.ID, description="Поле сортировки"),
        sort_order: SortOrder = Query(SortOrder.ASC, description="Порядок сортировки")
    ):
        self.cursor = cursor
        self.limit = limit
        self.sort_by = sort_by
        self.sort_order = sort_order


class FilterParams:
    """Параметры фильтрации."""
    
    def __init__(
        self,
        category: Optional[str] = Query(None, description="Категория"),
        min_price: Optional[float] = Query(None, ge=0, description="Минимальная цена"),
        max_price: Optional[float] = Query(None, ge=0, description="Максимальная цена"),
        min_stock: Optional[int] = Query(None, ge=0, description="Минимальный остаток"),
        is_active: Optional[bool] = Query(None, description="Только активные"),
        search: Optional[str] = Query(None, min_length=1, description="Поиск по названию/описанию"),
        created_after: Optional[datetime] = Query(None, description="Создано после"),
        created_before: Optional[datetime] = Query(None, description="Создано до")
    ):
        self.category = category
        self.min_price = min_price
        self.max_price = max_price
        self.min_stock = min_stock
        self.is_active = is_active
        self.search = search
        self.created_after = created_after
        self.created_before = created_before


# ============================================================
# ПРОМЕЖУТОЧНЫЙ СЛОЙ ДЛЯ ЗАПРОСОВ
# ============================================================

class ProductRepository:
    """Репозиторий для работы с товарами."""
    
    def __init__(self, db: Session):
        self.db = db
    
    def _apply_filters(self, query, filters: FilterParams):
        """Применение фильтров к запросу."""
        if filters.category:
            query = query.filter(ProductDB.category == filters.category)
        
        if filters.min_price is not None:
            query = query.filter(ProductDB.price >= filters.min_price)
        
        if filters.max_price is not None:
            query = query.filter(ProductDB.price <= filters.max_price)
        
        if filters.min_stock is not None:
            query = query.filter(ProductDB.stock >= filters.min_stock)
        
        if filters.is_active is not None:
            query = query.filter(ProductDB.is_active == filters.is_active)
        
        if filters.search:
            search_term = f"%{filters.search}%"
            query = query.filter(
                (ProductDB.name.contains(filters.search)) |
                (ProductDB.description.contains(filters.search))
            )
        
        if filters.created_after:
            query = query.filter(ProductDB.created_at >= filters.created_after)
        
        if filters.created_before:
            query = query.filter(ProductDB.created_at <= filters.created_before)
        
        return query
    
    def _apply_sorting(self, query, sort_by: SortField, sort_order: SortOrder):
        """Применение сортировки."""
        column = getattr(ProductDB, sort_by.value)
        
        if sort_order == SortOrder.ASC:
            query = query.order_by(column.asc())
        else:
            query = query.order_by(column.desc())
        
        return query
    
    def get_paginated(
        self,
        pagination: PaginationParams,
        filters: FilterParams
    ) -> tuple[List[ProductDB], int]:
        """Получение товаров с пагинацией (offset/limit)."""
        query = self.db.query(ProductDB)
        
        # Применение фильтров
        query = self._apply_filters(query, filters)
        
        # Подсчёт общего количества
        total = query.count()
        
        # Сортировка
        query = self._apply_sorting(query, pagination.sort_by, pagination.sort_order)
        
        # Пагинация
        query = query.offset(pagination.offset).limit(pagination.limit)
        
        return query.all(), total
    
    def get_cursor_paginated(
        self,
        cursor_params: CursorParams,
        filters: FilterParams
    ) -> tuple[List[ProductDB], Optional[str], bool]:
        """Получение товаров с курсорной пагинацией."""
        query = self.db.query(ProductDB)
        
        # Применение фильтров
        query = self._apply_filters(query, filters)
        
        # Применение курсора
        if cursor_params.cursor:
            try:
                cursor_value = int(cursor_params.cursor)
                column = getattr(ProductDB, cursor_params.sort_by.value)
                
                if cursor_params.sort_order == SortOrder.ASC:
                    query = query.filter(column > cursor_value)
                else:
                    query = query.filter(column < cursor_value)
            except (ValueError, TypeError):
                pass
        
        # Сортировка
        query = self._apply_sorting(query, cursor_params.sort_by, cursor_params.sort_order)
        
        # Получаем на один товар больше, чтобы проверить наличие следующей страницы
        items = query.limit(cursor_params.limit + 1).all()
        
        has_more = len(items) > cursor_params.limit
        if has_more:
            items = items[:-1]
        
        # Формируем следующий курсор
        next_cursor = None
        if has_more and items:
            last_item = items[-1]
            cursor_value = getattr(last_item, cursor_params.sort_by.value)
            next_cursor = str(cursor_value)
        
        return items, next_cursor, has_more
    
    def get_categories(self) -> List[str]:
        """Получение списка категорий."""
        categories = self.db.query(ProductDB.category).distinct().all()
        return [c[0] for c in categories]


# ============================================================
# FASTAPI ПРИЛОЖЕНИЕ
# ============================================================

app = FastAPI(title="Products API", description="API с пагинацией и фильтрацией")


# Dependency для получения сессии
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


# Получение репозитория
def get_repository(db: Session = Depends(get_db)):
    return ProductRepository(db)


@app.post("/products/generate")
def generate_test_data(
    count: int = Query(100, ge=1, le=1000),
    db: Session = Depends(get_db)
):
    """Генерация тестовых данных."""
    import random
    
    categories = ["electronics", "clothing", "books", "sports", "toys", "home"]
    names = [
        "Ноутбук", "Смартфон", "Наушники", "Клавиатура", "Мышь", "Монитор",
        "Футболка", "Джинсы", "Куртка", "Кроссовки", "Книга", "Игрушка"
    ]
    
    for i in range(count):
        product = ProductDB(
            name=f"{random.choice(names)} {i+1}",
            description=f"Описание товара {i+1}",
            price=random.uniform(100, 10000),
            category=random.choice(categories),
            stock=random.randint(0, 100),
            is_active=random.choice([True, False]),
            views=random.randint(0, 1000)
        )
        db.add(product)
    
    db.commit()
    return {"message": f"Сгенерировано {count} товаров"}


@app.get("/products", response_model=ProductListResponse)
def get_products(
    pagination: PaginationParams = Depends(),
    filters: FilterParams = Depends(),
    repo: ProductRepository = Depends(get_repository)
):
    """
    Получение списка товаров с пагинацией и фильтрацией.
    
    Поддерживает:
    - Пагинацию (page, limit)
    - Сортировку (sort_by, sort_order)
    - Фильтрацию по категории, цене, остатку
    - Поиск по названию/описанию
    - Фильтрацию по дате создания
    """
    items, total = repo.get_paginated(pagination, filters)
    
    pages = math.ceil(total / pagination.limit) if total > 0 else 1
    
    return ProductListResponse(
        data=[ProductResponse.model_validate(item) for item in items],
        total=total,
        page=pagination.page,
        limit=pagination.limit,
        pages=pages,
        has_next=pagination.page < pages,
        has_prev=pagination.page > 1
    )


@app.get("/products/cursor", response_model=CursorResponse)
def get_products_cursor(
    cursor_params: CursorParams = Depends(),
    filters: FilterParams = Depends(),
    repo: ProductRepository = Depends(get_repository)
):
    """
    Получение списка товаров с курсорной пагинацией.
    
    Преимущества курсорной пагинации:
    - Высокая производительность при больших данных
    - Стабильные результаты при добавлении новых записей
    
    Недостатки:
    - Нельзя перейти на произвольную страницу
    - Нужно хранить курсор
    """
    items, next_cursor, has_more = repo.get_cursor_paginated(cursor_params, filters)
    
    return CursorResponse(
        data=[ProductResponse.model_validate(item) for item in items],
        next_cursor=next_cursor,
        has_more=has_more,
        limit=cursor_params.limit
    )


@app.get("/products/categories")
def get_categories(repo: ProductRepository = Depends(get_repository)):
    """Получение списка категорий для фильтрации."""
    return {"categories": repo.get_categories()}


@app.get("/products/stats")
def get_stats(repo: ProductRepository = Depends(get_repository)):
    """Статистика по товарам."""
    db = repo.db
    
    total = db.query(func.count(ProductDB.id)).scalar()
    avg_price = db.query(func.avg(ProductDB.price)).scalar()
    total_stock = db.query(func.sum(ProductDB.stock)).scalar()
    
    return {
        "total_products": total,
        "average_price": round(avg_price, 2) if avg_price else 0,
        "total_stock": total_stock or 0
    }


@app.get("/products/{product_id}", response_model=ProductResponse)
def get_product(product_id: int, repo: ProductRepository = Depends(get_repository)):
    """Получение товара по ID с увеличением счётчика просмотров."""
    db = repo.db
    
    product = db.query(ProductDB).filter(ProductDB.id == product_id).first()
    if not product:
        raise HTTPException(status_code=404, detail="Product not found")
    
    # Увеличиваем счётчик просмотров
    product.views += 1
    db.commit()
    
    return ProductResponse.model_validate(product)


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000, reload=True)
Тестирование API
bash
# Генерация тестовых данных
curl -X POST "http://localhost:8000/products/generate?count=100"

# Получение списка с пагинацией
curl -X GET "http://localhost:8000/products?page=2&limit=10"

# Сортировка по цене (возрастание)
curl -X GET "http://localhost:8000/products?sort_by=price&sort_order=asc"

# Фильтрация по категории
curl -X GET "http://localhost:8000/products?category=electronics"

# Фильтрация по диапазону цен
curl -X GET "http://localhost:8000/products?min_price=500&max_price=2000"

# Поиск по названию
curl -X GET "http://localhost:8000/products?search=ноутбук"

# Комбинированная фильтрация
curl -X GET "http://localhost:8000/products?category=electronics&min_price=1000&is_active=true&sort_by=price&sort_order=desc"

# Курсорная пагинация (первая страница)
curl -X GET "http://localhost:8000/products/cursor?limit=10"

# Курсорная пагинация (следующая страница)
curl -X GET "http://localhost:8000/products/cursor?cursor=100&limit=10"

# Категории
curl -X GET "http://localhost:8000/products/categories"

# Статистика
curl -X GET "http://localhost:8000/products/stats"

# Получение товара по ID
curl -X GET "http://localhost:8000/products/1"
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (простая пагинация page/limit)

Варианты 9-17: средний (+ фильтрация, сортировка)

Варианты 18-25: сложный (курсорная пагинация, полнотекстовый поиск)

Варианты 1-8 (Базовый уровень)
№	Ресурс	Пагинация	Сортировка
1	Пользователи	page/limit	по ID
2	Посты	page/limit	по дате
3	Комментарии	page/limit	по ID
4	Задачи	page/limit	по приоритету
5	Заказы	page/limit	по дате
6	Продукты	page/limit	по цене
7	События	page/limit	по дате
8	Файлы	page/limit	по имени
Варианты 9-17 (Средний уровень)
№	Ресурс	Фильтрация	Сортировка
9	Пользователи	по статусу, роли	по имени, дате
10	Посты	по автору, статусу	по дате, просмотрам
11	Товары	по категории, цене	по цене, популярности
12	Заказы	по статусу, сумме	по дате, сумме
13	Клиенты	по городу, статусу	по имени, дате
14	Задачи	по статусу, приоритету	по дедлайну
15	Сотрудники	по отделу, должности	по зарплате
16	Студенты	по группе, курсу	по среднему баллу
17	Транзакции	по типу, сумме	по дате, сумме
Варианты 18-25 (Сложный уровень)
№	Ресурс	Дополнительно
18	Товары	Курсорная пагинация
19	Посты	Полнотекстовый поиск
20	Логи	Фильтрация по диапазону дат
21	События	Временная шкала
22	Мессенджер	Пагинация по чатам
23	Новости	Поиск по тегам
24	Аналитика	Агрегация по периодам
25	Файлы	Поиск по метаданным
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Пагинация не работает
3 (удовлетворительно)	Реализована page/limit пагинация
4 (хорошо)	+ фильтрация, сортировка
5 (отлично)	+ курсорная пагинация, метаинформация
5. Шпаргалка
python
# === OFFSET/LIMIT ===
offset = (page - 1) * limit
items = db.query(Model).offset(offset).limit(limit).all()
total = db.query(Model).count()

# === ФИЛЬТРАЦИЯ ===
query = db.query(Model)
if category:
    query = query.filter(Model.category == category)
if min_price:
    query = query.filter(Model.price >= min_price)

# === СОРТИРОВКА ===
if sort_order == "asc":
    query = query.order_by(getattr(Model, sort_by).asc())
else:
    query = query.order_by(getattr(Model, sort_by).desc())

# === МЕТАИНФОРМАЦИЯ ===
response = {
    "data": items,
    "total": total,
    "page": page,
    "limit": limit,
    "pages": (total + limit - 1) // limit
}

# === КУРСОРНАЯ ПАГИНАЦИЯ ===
if cursor:
    query = query.filter(Model.id > int(cursor))
items = query.limit(limit + 1).all()
has_more = len(items) > limit
next_cursor = str(items[-1].id) if has_more else None
Карточка студента
text
ПЗ 2.51. РЕАЛИЗАЦИЯ ПАГИНАЦИИ И ФИЛЬТРАЦИИ В REST API

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== ПАГИНАЦИЯ ===

□ page/limit (страничная)
□ cursor (курсорная)
□ Метаинформация (total, pages)

=== ФИЛЬТРАЦИЯ ===

□ Точное совпадение
□ Диапазон (min/max)
□ Поиск (search)
□ Множественный выбор
□ По дате

=== СОРТИРОВКА ===

□ По одному полю
□ По нескольким полям
□ ASC/DESC

=== ОТЧЁТ ===

Количество эндпоинтов: _____
Время ответа (p95): _____ мс

Дата выполнения: _____________
