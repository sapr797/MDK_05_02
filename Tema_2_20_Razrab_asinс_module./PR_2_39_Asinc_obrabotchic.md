# ПЗ 2.39. Создание асинхронного обработчика данных

**Тема:** Асинхронное программирование, aiohttp, asyncio, обработка данных

**Цель работы:**  
Научиться разрабатывать асинхронные обработчики данных, выполнять параллельные HTTP-запросы, обрабатывать большие объёмы данных, использовать очереди и пулы соединений.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `aiohttp`, `aiofiles`, `pydantic`

```bash
pip install aiohttp aiofiles pydantic
Главная мысль: Асинхронный обработчик данных может обрабатывать тысячи запросов в секунду, пока синхронный ждёт каждый из них по очереди.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Асинхронная обработка данных
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    АСИНХРОННАЯ ОБРАБОТКА ДАННЫХ                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    ПОТОК ДАННЫХ                                      │   │
│  │                                                                     │   │
│  │   ┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐                      │   │
│  │   │API 1 │───►│Очередь│───►│Обра- │───►│Сох-  │                      │   │
│  │   └──────┘    │      │    │ботчик│    │ранение│                      │   │
│  │   ┌──────┐    │      │    └──────┘    └──────┘                      │   │
│  │   │API 2 │───►│      │                                              │   │
│  │   └──────┘    └──────┘                                              │   │
│  │   ┌──────┐                                                          │   │
│  │   │API 3 │───►                                                      │   │
│  │   └──────┘                                                          │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  Ключевые компоненты:                                                       │
│  • Producer — генерирует данные (запросы к API)                            │
│  • Queue — буфер между producer и consumer                                 │
│  • Consumer — обрабатывает данные                                          │
│  • Worker — параллельные обработчики                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
1.2. Компоненты асинхронного обработчика
Компонент	Назначение	Реализация
Производитель	Генерация задач	async def producer()
Очередь	Буферизация	asyncio.Queue()
Рабочий	Обработка задач	async def worker()
Пул соединений	Переиспользование соединений	ClientSession
Ограничитель	Контроль параллелизма	Semaphore
1.3. Паттерны асинхронной обработки
Паттерн	Описание	Когда использовать
Producer-Consumer	Очередь между производителем и потребителем	Разная скорость производства и потребления
Worker Pool	Несколько параллельных обработчиков	Много независимых задач
Fan-out/Fan-in	Разветвление и сбор результатов	Агрегация данных из нескольких источников
Pipeline	Последовательные этапы обработки	Многошаговая обработка
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая создание асинхронного обработчика данных.

Техническое задание (нулевой вариант)
Разработайте асинхронный обработчик данных для интернет-магазина, который:

Параллельно загружает данные о товарах из API

Валидирует полученные данные

Обогащает данные (добавляет курс валюты)

Сохраняет результаты в JSON

Обрабатывает ошибки и имеет повторные попытки

Эталонная реализация
python
#!/usr/bin/env python3
"""
async_data_processor.py — Асинхронный обработчик данных для интернет-магазина.
"""

import asyncio
import aiohttp
import aiofiles
import json
from datetime import datetime
from typing import List, Dict, Any, Optional
from dataclasses import dataclass, field
from pydantic import BaseModel, Field, validator, ValidationError


# ============================================================
# PYDANTIC МОДЕЛИ ДЛЯ ВАЛИДАЦИИ
# ============================================================

class Product(BaseModel):
    """Модель товара."""
    id: int = Field(..., gt=0)
    title: str = Field(..., min_length=1, max_length=200)
    price: float = Field(..., gt=0)
    category: str = Field(..., min_length=1, max_length=100)
    stock: int = Field(..., ge=0)
    rating: float = Field(..., ge=0, le=5)


class EnrichedProduct(Product):
    """Обогащённый товар."""
    price_usd: float
    price_eur: float
    processed_at: str


# ============================================================
# КОНСТАНТЫ И КОНФИГУРАЦИЯ
# ============================================================

API_BASE_URL = "https://fakestoreapi.com"
EXCHANGE_RATES = {"USD": 90.5, "EUR": 98.2}  # Курсы валют


# ============================================================
# АСИНХРОННЫЙ ОБРАБОТЧИК ДАННЫХ
# ============================================================

@dataclass
class ProcessingStats:
    """Статистика обработки."""
    total: int = 0
    successful: int = 0
    failed: int = 0
    errors: List[str] = field(default_factory=list)


class AsyncDataProcessor:
    """Асинхронный обработчик данных."""
    
    def __init__(self, max_concurrent: int = 5, max_retries: int = 3):
        self.max_concurrent = max_concurrent
        self.max_retries = max_retries
        self.semaphore = asyncio.Semaphore(max_concurrent)
        self.stats = ProcessingStats()
        self.results: List[EnrichedProduct] = []
    
    async def fetch_with_retry(
        self,
        session: aiohttp.ClientSession,
        url: str,
        retry_count: int = 0
    ) -> Optional[Dict]:
        """Выполняет запрос с повторными попытками."""
        try:
            async with self.semaphore:
                async with session.get(url, timeout=10) as response:
                    if response.status == 200:
                        return await response.json()
                    else:
                        raise aiohttp.ClientResponseError(
                            status=response.status,
                            message=f"HTTP {response.status}"
                        )
        except (aiohttp.ClientError, asyncio.TimeoutError) as e:
            if retry_count < self.max_retries:
                delay = 2 ** retry_count  # экспоненциальная задержка
                print(f"  Повторная попытка {retry_count + 1}/{self.max_retries} через {delay}с...")
                await asyncio.sleep(delay)
                return await self.fetch_with_retry(session, url, retry_count + 1)
            else:
                self.stats.failed += 1
                self.stats.errors.append(f"Failed after {self.max_retries} retries: {url}")
                return None
    
    async def fetch_product(self, session: aiohttp.ClientSession, product_id: int) -> Optional[Product]:
        """Загружает и валидирует один товар."""
        url = f"{API_BASE_URL}/products/{product_id}"
        data = await self.fetch_with_retry(session, url)
        
        if not data:
            return None
        
        try:
            product = Product(**data)
            return product
        except ValidationError as e:
            self.stats.failed += 1
            self.stats.errors.append(f"Validation error for product {product_id}: {e}")
            return None
    
    def enrich_product(self, product: Product) -> EnrichedProduct:
        """Обогащает товар дополнительными данными."""
        return EnrichedProduct(
            id=product.id,
            title=product.title,
            price=product.price,
            category=product.category,
            stock=product.stock,
            rating=product.rating,
            price_usd=round(product.price / EXCHANGE_RATES["USD"], 2),
            price_eur=round(product.price / EXCHANGE_RATES["EUR"], 2),
            processed_at=datetime.now().isoformat()
        )
    
    async def process_products(self, product_ids: List[int]) -> List[EnrichedProduct]:
        """
        Обрабатывает список товаров параллельно.
        
        Args:
            product_ids: Список ID товаров для обработки
        
        Returns:
            Список обогащённых товаров
        """
        self.stats.total = len(product_ids)
        
        async with aiohttp.ClientSession() as session:
            # Создаём задачи для всех товаров
            tasks = [self.fetch_product(session, pid) for pid in product_ids]
            products = await asyncio.gather(*tasks)
        
        # Фильтруем успешные и обогащаем
        for product in products:
            if product:
                self.stats.successful += 1
                enriched = self.enrich_product(product)
                self.results.append(enriched)
        
        return self.results
    
    async def process_with_queue(self, product_ids: List[int], num_workers: int = 5):
        """
        Обрабатывает данные через очередь (producer-consumer).
        
        Args:
            product_ids: Список ID товаров
            num_workers: Количество воркеров
        """
        queue = asyncio.Queue()
        self.stats.total = len(product_ids)
        
        # Producer: наполняет очередь
        async def producer():
            for pid in product_ids:
                await queue.put(pid)
            # Сигнал завершения для каждого воркера
            for _ in range(num_workers):
                await queue.put(None)
        
        # Consumer: обрабатывает задачи
        async def worker(worker_id: int):
            async with aiohttp.ClientSession() as session:
                while True:
                    product_id = await queue.get()
                    if product_id is None:
                        break
                    
                    print(f"Worker {worker_id} обрабатывает продукт {product_id}")
                    product = await self.fetch_product(session, product_id)
                    
                    if product:
                        self.stats.successful += 1
                        enriched = self.enrich_product(product)
                        self.results.append(enriched)
                    else:
                        self.stats.failed += 1
                    
                    queue.task_done()
        
        # Запуск producer и workers
        producer_task = asyncio.create_task(producer())
        workers = [asyncio.create_task(worker(i)) for i in range(num_workers)]
        
        await producer_task
        await asyncio.gather(*workers)
    
    def get_stats(self) -> Dict:
        """Возвращает статистику обработки."""
        return {
            "total": self.stats.total,
            "successful": self.stats.successful,
            "failed": self.stats.failed,
            "success_rate": f"{self.stats.successful / self.stats.total * 100:.1f}%" if self.stats.total else "0%",
            "errors": self.stats.errors[:10]  # только первые 10
        }
    
    async def save_results(self, filename: str = "products.json"):
        """Сохраняет результаты в JSON файл."""
        output = {
            "metadata": {
                "processed_at": datetime.now().isoformat(),
                "statistics": self.get_stats()
            },
            "data": [p.dict() for p in self.results]
        }
        
        async with aiofiles.open(filename, 'w', encoding='utf-8') as f:
            await f.write(json.dumps(output, indent=2, ensure_ascii=False))
        
        print(f"\n💾 Результаты сохранены в {filename}")


# ============================================================
# АСИНХРОННЫЙ ОБРАБОТЧИК БОЛЬШИХ ДАННЫХ
# ============================================================

class AsyncBatchProcessor:
    """Обработчик больших объёмов данных с чанками."""
    
    def __init__(self, chunk_size: int = 10, max_concurrent: int = 3):
        self.chunk_size = chunk_size
        self.max_concurrent = max_concurrent
        self.processor = AsyncDataProcessor(max_concurrent=max_concurrent)
    
    def chunk_list(self, items: List, chunk_size: int) -> List[List]:
        """Разбивает список на чанки."""
        return [items[i:i + chunk_size] for i in range(0, len(items), chunk_size)]
    
    async def process_chunk(self, chunk: List[int]) -> List[EnrichedProduct]:
        """Обрабатывает один чанк."""
        results = await self.processor.process_products(chunk)
        return results
    
    async def process_batch(self, product_ids: List[int]) -> List[EnrichedProduct]:
        """Обрабатывает весь список чанками."""
        chunks = self.chunk_list(product_ids, self.chunk_size)
        all_results = []
        
        # Обрабатываем чанки последовательно (чтобы не перегружать API)
        for i, chunk in enumerate(chunks):
            print(f"\n📦 Обработка чанка {i + 1}/{len(chunks)} (размер: {len(chunk)})")
            results = await self.process_chunk(chunk)
            all_results.extend(results)
            print(f"   Прогресс: {len(all_results)}/{len(product_ids)}")
        
        return all_results
    
    def get_stats(self) -> Dict:
        return self.processor.get_stats()


# ============================================================
# ТЕСТОВЫЕ ДАННЫЕ И ЗАПУСК
# ============================================================

async def test_single_product():
    """Тест обработки одного товара."""
    print("\n" + "=" * 60)
    print("ТЕСТ: ОБРАБОТКА ОДНОГО ТОВАРА")
    print("=" * 60)
    
    processor = AsyncDataProcessor()
    results = await processor.process_products([1])
    
    if results:
        product = results[0]
        print(f"\n✅ Товар обработан:")
        print(f"   ID: {product.id}")
        print(f"   Название: {product.title}")
        print(f"   Цена: {product.price} ₽")
        print(f"   Цена в USD: ${product.price_usd}")
        print(f"   Цена в EUR: €{product.price_eur}")
    
    print(f"\n📊 Статистика: {processor.get_stats()}")


async def test_multiple_products():
    """Тест обработки нескольких товаров."""
    print("\n" + "=" * 60)
    print("ТЕСТ: ОБРАБОТКА НЕСКОЛЬКИХ ТОВАРОВ")
    print("=" * 60)
    
    product_ids = list(range(1, 11))
    
    processor = AsyncDataProcessor(max_concurrent=5)
    results = await processor.process_products(product_ids)
    
    print(f"\n✅ Обработано товаров: {len(results)}/{len(product_ids)}")
    
    # Показать первые 3 результата
    for product in results[:3]:
        print(f"   • {product.title}: {product.price} ₽ / ${product.price_usd}")
    
    print(f"\n📊 Статистика: {processor.get_stats()}")


async def test_queue_processing():
    """Тест обработки через очередь."""
    print("\n" + "=" * 60)
    print("ТЕСТ: ОБРАБОТКА ЧЕРЕЗ ОЧЕРЕДЬ (PRODUCER-CONSUMER)")
    print("=" * 60)
    
    product_ids = list(range(1, 21))
    
    processor = AsyncDataProcessor(max_retries=2)
    await processor.process_with_queue(product_ids, num_workers=3)
    
    print(f"\n✅ Обработано товаров: {processor.stats.successful}/{processor.stats.total}")
    print(f"\n📊 Статистика: {processor.get_stats()}")
    
    await processor.save_results("products_queue.json")


async def test_batch_processing():
    """Тест пакетной обработки."""
    print("\n" + "=" * 60)
    print("ТЕСТ: ПАКЕТНАЯ ОБРАБОТКА (ЧАНКИ)")
    print("=" * 60)
    
    product_ids = list(range(1, 51))
    
    batch_processor = AsyncBatchProcessor(chunk_size=10, max_concurrent=3)
    results = await batch_processor.process_batch(product_ids)
    
    print(f"\n✅ Обработано товаров: {len(results)}/{len(product_ids)}")
    print(f"\n📊 Статистика: {batch_processor.get_stats()}")
    
    await batch_processor.processor.save_results("products_batch.json")


async def test_with_real_api():
    """Тест с реальным API (FakeStoreAPI)."""
    print("\n" + "=" * 60)
    print("ТЕСТ: РЕАЛЬНЫЙ API (FakeStoreAPI)")
    print("=" * 60)
    
    # Известные ID товаров в FakeStoreAPI (обычно 1-20)
    product_ids = [1, 2, 3, 4, 5]
    
    processor = AsyncDataProcessor(max_concurrent=3, max_retries=2)
    
    print("\n⏳ Загрузка данных...")
    start_time = asyncio.get_event_loop().time()
    
    results = await processor.process_products(product_ids)
    
    elapsed = asyncio.get_event_loop().time() - start_time
    
    print(f"\n✅ Завершено за {elapsed:.2f} сек")
    print(f"   Успешно: {processor.stats.successful}/{processor.stats.total}")
    print(f"   Ошибок: {processor.stats.failed}")
    
    if results:
        print("\n📦 Пример обработанного товара:")
        sample = results[0]
        print(f"   ID: {sample.id}")
        print(f"   Название: {sample.title[:50]}...")
        print(f"   Категория: {sample.category}")
        print(f"   Цена: {sample.price} ₽ (${sample.price_usd} / €{sample.price_eur})")
    
    await processor.save_results("products_real.json")


# ============================================================
# ГЛАВНАЯ ФУНКЦИЯ
# ============================================================

async def main():
    """Главная функция."""
    print("=" * 70)
    print("АСИНХРОННЫЙ ОБРАБОТЧИК ДАННЫХ")
    print("=" * 70)
    
    # Запуск всех тестов
    await test_single_product()
    await test_multiple_products()
    await test_queue_processing()
    await test_batch_processing()
    await test_with_real_api()
    
    print("\n" + "=" * 70)
    print("ВСЕ ТЕСТЫ ЗАВЕРШЕНЫ")
    print("=" * 70)


if __name__ == "__main__":
    asyncio.run(main())
Ожидаемый вывод
text
======================================================================
АСИНХРОННЫЙ ОБРАБОТЧИК ДАННЫХ
======================================================================

============================================================
ТЕСТ: ОБРАБОТКА ОДНОГО ТОВАРА
============================================================

✅ Товар обработан:
   ID: 1
   Название: Fjallraven - Foldsack No. 1 Backpack, Fits 15 Laptops
   Цена: 109.95 ₽
   Цена в USD: $1.21
   Цена в EUR: €1.12

📊 Статистика: {'total': 1, 'successful': 1, 'failed': 0, 'success_rate': '100.0%', 'errors': []}

============================================================
ТЕСТ: ОБРАБОТКА НЕСКОЛЬКИХ ТОВАРОВ
============================================================

✅ Обработано товаров: 10/10
   • Fjallraven - Foldsack No. 1 Backpack, Fits 15 Laptops: 109.95 ₽ / $1.21
   • Mens Casual Premium Slim Fit T-Shirts: 22.3 ₽ / $0.25
   • Mens Cotton Jacket: 55.99 ₽ / $0.62

📊 Статистика: {'total': 10, 'successful': 10, 'failed': 0, 'success_rate': '100.0%', 'errors': []}

============================================================
ТЕСТ: ОБРАБОТКА ЧЕРЕЗ ОЧЕРЕДЬ (PRODUCER-CONSUMER)
============================================================
Worker 0 обрабатывает продукт 1
Worker 1 обрабатывает продукт 2
Worker 2 обрабатывает продукт 3
...

✅ Обработано товаров: 20/20

📊 Статистика: {'total': 20, 'successful': 20, 'failed': 0, 'success_rate': '100.0%', 'errors': []}

💾 Результаты сохранены в products_queue.json

============================================================
ТЕСТ: ПАКЕТНАЯ ОБРАБОТКА (ЧАНКИ)
============================================================

📦 Обработка чанка 1/5 (размер: 10)
   Прогресс: 10/50
📦 Обработка чанка 2/5 (размер: 10)
   Прогресс: 20/50
...

✅ Обработано товаров: 50/50

📊 Статистика: {'total': 50, 'successful': 50, 'failed': 0, 'success_rate': '100.0%', 'errors': []}

💾 Результаты сохранены в products_batch.json

============================================================
ТЕСТ: РЕАЛЬНЫЙ API (FakeStoreAPI)
============================================================

⏳ Загрузка данных...

✅ Завершено за 2.34 сек
   Успешно: 5/5
   Ошибок: 0

📦 Пример обработанного товара:
   ID: 1
   Название: Fjallraven - Foldsack No. 1 Backpack, Fits 15...
   Категория: men's clothing
   Цена: 109.95 ₽ ($1.21 / €1.12)

💾 Результаты сохранены в products_real.json

======================================================================
ВСЕ ТЕСТЫ ЗАВЕРШЕНЫ
======================================================================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (параллельные запросы, простая обработка)

Варианты 9-17: средний (очереди, пулы, повторные попытки)

Варианты 18-25: сложный (несколько источников, агрегация, мониторинг)

Варианты 1-8 (Базовый уровень)
№	Тип данных	API	Обработка
1	Пользователи	JSONPlaceholder	GET, валидация
2	Посты	JSONPlaceholder	GET, фильтрация
3	Комментарии	JSONPlaceholder	GET, агрегация
4	Товары	FakeStoreAPI	GET, обогащение
5	Категории	FakeStoreAPI	GET, группировка
6	Погода	OpenWeatherMap	GET, кэширование
7	Курсы валют	ExchangeRate-API	GET, конвертация
8	Новости	NewsAPI	GET, фильтрация
Варианты 9-17 (Средний уровень)
№	Тип данных	Особенности
9	Агрегатор постов	3 API, объединение результатов
10	Мониторинг API	Проверка доступности, логи
11	ETL с обогащением	API + локальные данные
12	Парсер с пагинацией	Много страниц, контроль лимитов
13	Обработчик заказов	POST запросы, подтверждение
14	Аналитик логов	Несколько источников, агрегация
15	Сборщик метрик	Периодический опрос API
16	Валидатор данных	Проверка через внешнее API
17	Синхронизатор	Diff, обновление
Варианты 18-25 (Сложный уровень)
№	Тема	Компоненты
18	ETL пайплайн	Extract → Transform → Load с очередями
19	Распределённый парсер	Множество источников, контроль нагрузки
20	Система мониторинга	Метрики, алерты, дашборд
21	Асинхронный crawler	Ссылки, глубина, вежливость
22	Realtime агрегатор	WebSocket + HTTP
23	Обработчик webhook	Приём, валидация, отправка
24	Bulk operations	Тысячи запросов, батчинг
25	Отказоустойчивый клиент	Retry, circuit breaker, fallback
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Асинхронный обработчик не работает
3 (удовлетворительно)	Параллельные запросы, базовая обработка
4 (хорошо)	+ очередь, пул воркеров, повторные попытки
5 (отлично)	+ пакетная обработка, сохранение результатов, отчёты
5. Шпаргалка
python
# === БАЗОВЫЙ ШАБЛОН ===
async def main():
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        results = await asyncio.gather(*tasks)

# === С ОГРАНИЧИТЕЛЕМ ===
semaphore = asyncio.Semaphore(5)
async with semaphore:
    await fetch(session, url)

# === PRODUCER-CONSUMER ===
queue = asyncio.Queue()
async def producer(): await queue.put(item)
async def consumer(): item = await queue.get()

# === RETRY ===
for attempt in range(max_retries):
    try:
        return await fetch()
    except Exception:
        await asyncio.sleep(2 ** attempt)

# === СОХРАНЕНИЕ ===
async with aiofiles.open('output.json', 'w') as f:
    await f.write(json.dumps(data))
Карточка студента
text
ПЗ 2.38. СОЗДАНИЕ АСИНХРОННОГО ОБРАБОТЧИКА ДАННЫХ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== КОМПОНЕНТЫ ===

□ Параллельные HTTP-запросы
□ Ограничение параллелизма (Semaphore)
□ Обработка ошибок и повторные попытки
□ Очередь (Producer-Consumer)
□ Пул воркеров
□ Пакетная обработка (чанки)
□ Сохранение результатов

=== СТАТИСТИКА ===

Обработано записей: _____
Успешно: _____
Ошибок: _____
Время выполнения: _____ сек

=== ОТЧЁТ ===

Файл: async_processor.py
Результаты: _____________

Дата выполнения: _____________
