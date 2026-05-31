# ПЗ 2.40. Асинхронное API на FastAPI

**Тема:** Асинхронное программирование, FastAPI, aiohttp, WebSocket, фоновые задачи

**Цель работы:**  
Научиться создавать асинхронные API на FastAPI с использованием async/await, WebSocket-соединений, фоновых задач и интеграцией с внешними API.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `fastapi`, `uvicorn`, `aiohttp`, `aiofiles`, `pydantic`

```bash
pip install fastapi uvicorn aiohttp aiofiles pydantic
Главная мысль: FastAPI изначально асинхронный. Это позволяет обрабатывать тысячи запросов в секунду и эффективно работать с внешними API, базами данных и WebSocket.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Асинхронность в FastAPI
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    АСИНХРОННЫЙ WEB-СЕРВЕР НА FASTAPI                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    ЗАПРОСЫ КЛИЕНТОВ                                  │   │
│  │                                                                     │   │
│  │   ┌──────┐    ┌──────┐    ┌──────────────────────────────────┐     │   │
│  │   │Запрос1│───►│      │    │     АСИНХРОННЫЙ EVENT LOOP       │     │   │
│  │   └──────┘    │      │    │                                  │     │   │
│  │   ┌──────┐    │      │    │  ┌─────────┐  ┌─────────┐       │     │   │
│  │   │Запрос2│───►│      │───►│  │Задача 1 │  │Задача 2 │       │     │   │
│  │   └──────┘    │FastAPI│    │  │(async)  │  │(async)  │       │     │   │
│  │   ┌──────┐    │      │    │  └────┬────┘  └────┬────┘       │     │   │
│  │   │Запрос3│───►│      │    │       │          │              │     │   │
│  │   └──────┘    └──────┘    │       ▼          ▼              │     │   │
│  │                           │  ┌─────────────────────────┐    │     │   │
│  │                           │  │   Внешние API / БД       │    │     │   │
│  │                           │  └─────────────────────────┘    │     │   │
│  │                           └──────────────────────────────────┘     │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  Ключевые особенности:                                                      │
│  • Каждый запрос не блокирует другие                                       │
│  • Во время ожидания I/O сервер обрабатывает другие запросы                │
│  • Поддержка WebSocket для двусторонней связи                              │
│  • Фоновые задачи для длительных операций                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
1.2. Синхронные vs Асинхронные эндпоинты
Тип	Объявление	Когда использовать
Синхронный	@app.get('/path')	Лёгкие вычисления, работа с локальными данными
Асинхронный	@app.get('/path') с async def	I/O операции, HTTP-запросы, работа с БД
python
# Синхронный эндпоинт (блокирующий)
@app.get('/sync')
def sync_endpoint():
    time.sleep(1)  # Блокирует весь сервер!
    return {"status": "ok"}

# Асинхронный эндпоинт (неблокирующий)
@app.get('/async')
async def async_endpoint():
    await asyncio.sleep(1)  # Не блокирует другие запросы
    return {"status": "ok"}
1.3. Компоненты асинхронного API
Компонент	Назначение
Async эндпоинты	Обработка запросов без блокировки
BackgroundTasks	Фоновые задачи после ответа клиенту
WebSocket	Двусторонняя связь реального времени
Dependency Injection	Асинхронные зависимости
External API	Взаимодействие с внешними сервисами
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая создание асинхронного API на FastAPI.

Техническое задание (нулевой вариант)
Разработайте асинхронное API для агрегатора данных, которое:

Принимает список ID товаров и параллельно загружает данные из внешнего API

Валидирует полученные данные с помощью Pydantic

Кэширует результаты для повторных запросов

Поддерживает WebSocket для получения статуса обработки

Имеет фоновую задачу для периодического обновления данных

Эталонная реализация
python
#!/usr/bin/env python3
"""
async_fastapi.py — Асинхронное API на FastAPI.
"""

import asyncio
import aiohttp
import json
from datetime import datetime, timedelta
from typing import List, Dict, Any, Optional
from contextlib import asynccontextmanager

from fastapi import FastAPI, HTTPException, BackgroundTasks, WebSocket, WebSocketDisconnect, Depends, Query
from fastapi.responses import JSONResponse
from pydantic import BaseModel, Field, validator
import uvicorn


# ============================================================
# PYDANTIC МОДЕЛИ
# ============================================================

class ProductResponse(BaseModel):
    """Модель ответа с товаром."""
    id: int
    title: str
    price: float
    category: str
    rating: float
    fetched_at: str


class ProductsRequest(BaseModel):
    """Запрос на получение товаров."""
    product_ids: List[int] = Field(..., min_items=1, max_items=50)


# ============================================================
# КЭШ
# ============================================================

class AsyncCache:
    """Асинхронный кэш с TTL."""
    
    def __init__(self, default_ttl: int = 300):
        self._cache: Dict[str, tuple] = {}  # key: (data, expiry)
        self.default_ttl = default_ttl
    
    def _get_key(self, product_id: int) -> str:
        return f"product:{product_id}"
    
    def get(self, product_id: int) -> Optional[Dict]:
        key = self._get_key(product_id)
        if key in self._cache:
            data, expiry = self._cache[key]
            if datetime.now() < expiry:
                return data
            else:
                del self._cache[key]
        return None
    
    def set(self, product_id: int, data: Dict, ttl: Optional[int] = None):
        key = self._get_key(product_id)
        expiry = datetime.now() + timedelta(seconds=ttl or self.default_ttl)
        self._cache[key] = (data, expiry)
    
    def clear(self):
        self._cache.clear()


# ============================================================
# СОЗДАНИЕ ПРИЛОЖЕНИЯ
# ============================================================

@asynccontextmanager
async def lifespan(app: FastAPI):
    """Управление жизненным циклом приложения."""
    print("🚀 Запуск приложения...")
    app.state.cache = AsyncCache()
    app.state.session = aiohttp.ClientSession()
    yield
    print("🛑 Остановка приложения...")
    await app.state.session.close()


app = FastAPI(
    title="Async Product API",
    description="Асинхронное API для агрегации данных о товарах",
    version="1.0.0",
    lifespan=lifespan
)


# ============================================================
# ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ
# ============================================================

async def fetch_product(session: aiohttp.ClientSession, product_id: int) -> Optional[Dict]:
    """Загружает один товар из внешнего API."""
    url = f"https://fakestoreapi.com/products/{product_id}"
    
    try:
        async with session.get(url, timeout=10) as response:
            if response.status == 200:
                return await response.json()
            else:
                return None
    except (aiohttp.ClientError, asyncio.TimeoutError):
        return None


async def fetch_products_parallel(product_ids: List[int]) -> List[Dict]:
    """Параллельная загрузка товаров."""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_product(session, pid) for pid in product_ids]
        results = await asyncio.gather(*tasks)
        return [r for r in results if r is not None]


# ============================================================
# ОСНОВНЫЕ ЭНДПОИНТЫ
# ============================================================

@app.get("/")
async def root():
    """Корневой эндпоинт."""
    return {
        "message": "Async Product API",
        "endpoints": {
            "GET /products/{id}": "Получить один товар (с кэшем)",
            "POST /products/batch": "Получить несколько товаров (параллельно)",
            "GET /products/stats": "Статистика кэша",
            "WS /ws": "WebSocket для мониторинга"
        }
    }


@app.get("/products/{product_id}")
async def get_product(product_id: int = Path(..., gt=0, le=100)):
    """
    Получение одного товара с кэшированием.
    
    Args:
        product_id: ID товара (1-100)
    
    Returns:
        ProductResponse: Данные товара
    """
    # Проверка кэша
    cache = app.state.cache
    cached = cache.get(product_id)
    if cached:
        return {
            "id": product_id,
            "title": cached.get('title'),
            "price": cached.get('price'),
            "category": cached.get('category'),
            "rating": cached.get('rating', {}).get('rate', 0),
            "fetched_at": datetime.now().isoformat(),
            "from_cache": True
        }
    
    # Загрузка из API
    session = app.state.session
    data = await fetch_product(session, product_id)
    
    if not data:
        raise HTTPException(status_code=404, detail=f"Product {product_id} not found")
    
    # Сохранение в кэш
    cache.set(product_id, data)
    
    return {
        "id": product_id,
        "title": data.get('title'),
        "price": data.get('price'),
        "category": data.get('category'),
        "rating": data.get('rating', {}).get('rate', 0),
        "fetched_at": datetime.now().isoformat(),
        "from_cache": False
    }


@app.post("/products/batch")
async def get_products_batch(request: ProductsRequest):
    """
    Параллельное получение нескольких товаров.
    
    Args:
        request: Список ID товаров
    
    Returns:
        List[ProductResponse]: Список товаров
    """
    product_ids = request.product_ids
    
    # Загружаем параллельно
    start_time = asyncio.get_event_loop().time()
    results = await fetch_products_parallel(product_ids)
    elapsed = asyncio.get_event_loop().time() - start_time
    
    response = []
    for data in results:
        response.append({
            "id": data.get('id'),
            "title": data.get('title'),
            "price": data.get('price'),
            "category": data.get('category'),
            "rating": data.get('rating', {}).get('rate', 0),
            "fetched_at": datetime.now().isoformat()
        })
    
    return {
        "total": len(response),
        "requested": len(product_ids),
        "processing_time": round(elapsed, 3),
        "products": response
    }


@app.get("/products/stats")
async def get_cache_stats():
    """Статистика кэша."""
    cache = app.state.cache
    return {
        "cache_size": len(cache._cache),
        "cache_keys": list(cache._cache.keys())
    }


@app.delete("/products/cache")
async def clear_cache():
    """Очистка кэша."""
    app.state.cache.clear()
    return {"message": "Cache cleared"}


# ============================================================
# ФОНОВЫЕ ЗАДАЧИ
# ============================================================

async def refresh_products_background(product_ids: List[int], webhook_url: str = None):
    """
    Фоновая задача для обновления данных.
    
    Args:
        product_ids: Список ID для обновления
        webhook_url: URL для уведомления о завершении
    """
    await asyncio.sleep(2)  # Имитация работы
    
    results = await fetch_products_parallel(product_ids)
    
    # Обновляем кэш
    cache = app.state.cache
    for data in results:
        if data:
            cache.set(data.get('id'), data)
    
    # Отправка уведомления через webhook
    if webhook_url:
        try:
            async with aiohttp.ClientSession() as session:
                await session.post(webhook_url, json={
                    "status": "completed",
                    "updated": len(results),
                    "timestamp": datetime.now().isoformat()
                })
        except:
            pass


@app.post("/products/refresh")
async def refresh_products(
    product_ids: List[int] = Query(..., description="Список ID для обновления"),
    background_tasks: BackgroundTasks = BackgroundTasks(),
    webhook: Optional[str] = None
):
    """
    Запуск фонового обновления данных.
    
    Args:
        product_ids: Список ID товаров
        background_tasks: FastAPI BackgroundTasks
        webhook: URL для уведомления
    
    Returns:
        Ссылка на статус задачи
    """
    background_tasks.add_task(
        refresh_products_background,
        product_ids,
        webhook
    )
    
    return {
        "message": "Refresh started",
        "product_ids": product_ids,
        "status": "pending"
    }


# ============================================================
# WEBSOCKET
# ============================================================

class ConnectionManager:
    """Менеджер WebSocket соединений."""
    
    def __init__(self):
        self.active_connections: List[WebSocket] = []
    
    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)
    
    def disconnect(self, websocket: WebSocket):
        if websocket in self.active_connections:
            self.active_connections.remove(websocket)
    
    async def send_message(self, message: str):
        for connection in self.active_connections:
            try:
                await connection.send_text(message)
            except:
                pass
    
    async def broadcast(self, data: Dict):
        """Широковещательная рассылка."""
        for connection in self.active_connections:
            try:
                await connection.send_json(data)
            except:
                pass


manager = ConnectionManager()


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    """
    WebSocket эндпоинт для получения обновлений.
    
    Клиент может отправлять ID товаров и получать их данные.
    """
    await manager.connect(websocket)
    
    try:
        while True:
            # Получаем сообщение от клиента
            data = await websocket.receive_text()
            
            # Парсим ID товаров
            try:
                product_ids = json.loads(data)
                if isinstance(product_ids, list):
                    results = await fetch_products_parallel(product_ids)
                    
                    # Отправляем результаты
                    await websocket.send_json({
                        "type": "products",
                        "data": results,
                        "timestamp": datetime.now().isoformat()
                    })
                else:
                    await websocket.send_json({
                        "type": "error",
                        "message": "Expected list of product IDs"
                    })
            except json.JSONDecodeError:
                await websocket.send_json({
                    "type": "error",
                    "message": "Invalid JSON format"
                })
                
    except WebSocketDisconnect:
        manager.disconnect(websocket)


@app.get("/ws/broadcast")
async def broadcast_message(message: str):
    """Широковещательная рассылка через WebSocket."""
    await manager.broadcast({
        "type": "broadcast",
        "message": message,
        "timestamp": datetime.now().isoformat()
    })
    return {"message": "Broadcast sent"}


# ============================================================
# ОБРАБОТКА ОШИБОК
# ============================================================

@app.exception_handler(HTTPException)
async def http_exception_handler(request, exc):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": True,
            "code": exc.status_code,
            "message": exc.detail
        }
    )


# ============================================================
# ЗАПУСК
# ============================================================

if __name__ == "__main__":
    uvicorn.run(
        "async_fastapi:app",
        host="127.0.0.1",
        port=8000,
        reload=True
    )
Тестирование API
bash
# GET запрос (один товар)
curl -X GET http://localhost:8000/products/1

# POST запрос (пакет товаров)
curl -X POST http://localhost:8000/products/batch \
  -H "Content-Type: application/json" \
  -d '{"product_ids": [1, 2, 3, 4, 5]}'

# Запуск фонового обновления
curl -X POST "http://localhost:8000/products/refresh?product_ids=1&product_ids=2&product_ids=3"

# Статистика кэша
curl -X GET http://localhost:8000/products/stats

# Очистка кэша
curl -X DELETE http://localhost:8000/products/cache
WebSocket клиент (тестовый)
javascript
// client.html
const ws = new WebSocket('ws://localhost:8000/ws');

ws.onopen = () => {
    console.log('Connected');
    ws.send(JSON.stringify([1, 2, 3, 4, 5]));
};

ws.onmessage = (event) => {
    console.log('Received:', JSON.parse(event.data));
};
Ожидаемый вывод
text
INFO:     Started server process
INFO:     Waiting for application startup.
🚀 Запуск приложения...
INFO:     Application startup complete.

GET /products/1
Response: {"id":1,"title":"Fjallraven - Foldsack No. 1 Backpack...", "from_cache":false}

GET /products/1 (повторно)
Response: {"id":1,"title":"Fjallraven - Foldsack No. 1 Backpack...", "from_cache":true}

POST /products/batch
Response: {
  "total": 5,
  "requested": 5,
  "processing_time": 0.85,
  "products": [...]
}
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (асинхронные GET/POST эндпоинты)

Варианты 9-17: средний (+ кэш, фоновые задачи)

Варианты 18-25: сложный (+ WebSocket, внешние API)

Варианты 1-8 (Базовый уровень)
№	Тема API	Асинхронные эндпоинты
1	Пользователи	GET /users, GET /users/{id}
2	Посты	GET /posts, POST /posts
3	Комментарии	GET /comments, GET /comments/{id}
4	Товары	GET /products, GET /products/{id}
5	Категории	GET /categories, GET /categories/{id}
6	Заказы	GET /orders, POST /orders
7	События	GET /events, GET /events/{id}
8	Задачи	GET /tasks, POST /tasks
Варианты 9-17 (Средний уровень)
№	Тема API	Дополнительные возможности
9	Агрегатор постов	Кэш, параллельные запросы
10	Мониторинг API	Кэш, статистика
11	ETL API	Фоновые задачи
12	Парсер с пагинацией	Кэш страниц
13	Обработчик заказов	Фоновое подтверждение
14	Аналитик логов	Batch-обработка
15	Сборщик метрик	Периодическое обновление
16	Валидатор	Асинхронная проверка
17	Синхронизатор	Фоновое обновление
Варианты 18-25 (Сложный уровень)
№	Тема API	Компоненты
18	Агрегатор данных	Множество API, кэш, WebSocket
19	Чат-сервер	WebSocket, комнаты, сообщения
20	Система уведомлений	WebSocket, фоновые задачи
21	Realtime дашборд	WebSocket, статистика
22	Обработчик webhook	Асинхронный приём, обработка
23	API шлюз	Маршрутизация, кэш
24	Очередь задач	BackgroundTasks, статусы
25	Полнотекстовый поиск	Индексация, поиск
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	API не работает или не асинхронный
3 (удовлетворительно)	Базовые асинхронные эндпоинты
4 (хорошо)	+ кэш, фоновые задачи, обработка ошибок
5 (отлично)	+ WebSocket, параллельные запросы, мониторинг
5. Шпаргалка
python
# === АСИНХРОННЫЙ ЭНДПОИНТ ===
@app.get('/async')
async def async_endpoint():
    await asyncio.sleep(1)
    return {"data": "result"}

# === ФОНОВЫЕ ЗАДАЧИ ===
from fastapi import BackgroundTasks

@app.post('/task')
async def run_task(background_tasks: BackgroundTasks):
    background_tasks.add_task(long_running_task)
    return {"status": "started"}

# === WEBSOCKET ===
@app.websocket('/ws')
async def websocket(websocket: WebSocket):
    await websocket.accept()
    data = await websocket.receive_text()
    await websocket.send_json({"response": data})

# === КЭШ ===
from functools import lru_cache
from datetime import datetime, timedelta

class AsyncCache:
    def __init__(self, ttl: int = 60):
        self._cache = {}
        self.ttl = ttl
Карточка студента
text
ПЗ 2.39. АСИНХРОННОЕ API НА FASTAPI

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== КОМПОНЕНТЫ ===

□ Асинхронные эндпоинты (GET, POST)
□ Параллельные HTTP-запросы
□ Кэширование (асинхронный кэш)
□ Фоновые задачи (BackgroundTasks)
□ WebSocket соединения
□ Обработка ошибок

=== СТАТИСТИКА ===

Эндпоинтов: _____
Время ответа (среднее): _____ сек
Кэш: □ Да □ Нет
WebSocket: □ Да □ Нет

=== ОТЧЁТ ===

Файл: async_api.py
Документация: /docs

Дата выполнения: _____________
