# Тема 2.32. Асинхронное программирование в Python: async, await. Асинхронные HTTP-запросы с aiohttp

**Цель лекции:**  
Изучить основы асинхронного программирования в Python: ключевые слова `async` и `await`, понятие корутин, event loop. Научиться выполнять асинхронные HTTP-запросы с использованием библиотеки `aiohttp`.

> Главная мысль: **Асинхронность позволяет программе не ждать медленные операции (сеть, диск), а заниматься другими делами. Это увеличивает производительность в сотни раз.**

---

## Содержание

1. [Введение в асинхронное программирование](#1-введение-в-асинхронное-программирование)
2. [Синтаксис async/await](#2-синтаксис-asyncawait)
3. [Библиотека aiohttp](#3-библиотека-aiohttp)
4. [Асинхронные HTTP-запросы](#4-асинхронные-http-запросы)
5. [Обработка ошибок и таймауты](#5-обработка-ошибок-и-таймауты)
6. [Практические примеры](#6-практические-примеры)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Введение в асинхронное программирование

### 1.1. Синхронный vs Асинхронный подход
┌─────────────────────────────────────────────────────────────────────────────┐
│ СИНХРОННЫЙ vs АСИНХРОННЫЙ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ СИНХРОННЫЙ (последовательный): │
│ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ │
│ │Запрос1│───►│Запрос2│───►│Запрос3│───►│Готово│ │
│ └──────┘ └──────┘ └──────┘ └──────┘ │
│ ↓ ↓ ↓ │
│ Ожидание Ожидание Ожидание │
│ (1 сек) (1 сек) (1 сек) = 3 секунды │
│ │
│ АСИНХРОННЫЙ (параллельный): │
│ ┌──────┐ │
│ │Запрос1│──┐ │
│ └──────┘ │ ┌──────┐ │
│ ┌──────┐ ├──►Запрос2│───►Готово │
│ │Запрос2│──┘ └──────┘ │
│ └──────┘ │
│ ┌──────┐ │
│ │Запрос3│──────────────────────────────────────────────────────────────────┘│
│ └──────┘ │
│ ↓ │
│ Максимум = 1 секунда │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Ключевые понятия

| Понятие | Описание |
|---------|----------|
| **Event Loop** | Цикл событий, управляющий выполнением асинхронных задач |
| **Корутина (coroutine)** | Асинхронная функция, объявленная с `async def` |
| **Await** | Ожидание результата другой корутины |
| **Task** | Обёртка корутины для параллельного выполнения |

### 1.3. Когда нужна асинхронность

| Сценарий | Синхронно | Асинхронно |
|----------|-----------|------------|
| Много HTTP-запросов | Медленно | Очень быстро |
| WebSocket соединения | Сложно | Легко |
| Чат-боты | Сложно | Легко |
| Работа с БД (Async DB) | ❌ | ✅ |
| CPU-интенсивные задачи | ✅ | ❌ |

---

## 2. Синтаксис async/await

### 2.1. Определение корутин

```python
import asyncio


# Объявление асинхронной функции
async def hello():
    print("Hello")
    await asyncio.sleep(1)  # неблокирующая задержка
    print("World")


# Запуск корутины
asyncio.run(hello())  # Python 3.7+
2.2. Ожидание результата
python
import asyncio


async def fetch_data():
    print("Начинаем загрузку...")
    await asyncio.sleep(2)  # имитация сетевого запроса
    print("Загрузка завершена")
    return {"data": [1, 2, 3]}


async def main():
    print("Запуск main")
    result = await fetch_data()
    print(f"Получены данные: {result}")


asyncio.run(main())
2.3. Параллельное выполнение (Task)
python
import asyncio


async def task_one():
    await asyncio.sleep(2)
    print("Task 1 completed")
    return "Result 1"


async def task_two():
    await asyncio.sleep(1)
    print("Task 2 completed")
    return "Result 2"


async def main():
    # Параллельный запуск
    task1 = asyncio.create_task(task_one())
    task2 = asyncio.create_task(task_two())
    
    # Ожидание обоих
    results = await asyncio.gather(task1, task2)
    print(f"All results: {results}")


asyncio.run(main())
# Task 2 completed (через 1 сек)
# Task 1 completed (через 2 сек)
# All results: ['Result 1', 'Result 2']
2.4. gather vs wait
python
import asyncio


async def fetch(i: int):
    await asyncio.sleep(1)
    return f"Data {i}"


async def main_gather():
    # gather: возвращает список результатов
    results = await asyncio.gather(
        fetch(1), fetch(2), fetch(3)
    )
    print(f"Gather: {results}")


async def main_wait():
    # wait: возвращает кортеж (done, pending)
    tasks = [asyncio.create_task(fetch(i)) for i in range(1, 4)]
    done, pending = await asyncio.wait(tasks, return_when=asyncio.ALL_COMPLETED)
    results = [t.result() for t in done]
    print(f"Wait: {results}")


asyncio.run(main_gather())
2.5. Timeout и отмена задач
python
import asyncio


async def slow_task():
    await asyncio.sleep(5)
    return "Done"


async def main():
    task = asyncio.create_task(slow_task())
    
    try:
        # Ждём максимум 2 секунды
        result = await asyncio.wait_for(task, timeout=2)
        print(f"Результат: {result}")
    except asyncio.TimeoutError:
        print("Превышено время ожидания!")
        task.cancel()  # Отменяем задачу


asyncio.run(main())
3. Библиотека aiohttp
3.1. Установка и импорт
bash
pip install aiohttp
python
import aiohttp
import asyncio
3.2. Базовый GET запрос
python
import aiohttp
import asyncio


async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.text()


async def main():
    async with aiohttp.ClientSession() as session:
        html = await fetch_url(session, "https://httpbin.org/get")
        print(html[:200])  # первые 200 символов


asyncio.run(main())
3.3. Получение JSON
python
import aiohttp
import asyncio


async def fetch_json(session, url):
    async with session.get(url) as response:
        return await response.json()


async def main():
    async with aiohttp.ClientSession() as session:
        data = await fetch_json(session, "https://httpbin.org/json")
        print(data)


asyncio.run(main())
3.4. POST запрос
python
import aiohttp
import asyncio


async def post_data(session, url, data):
    async with session.post(url, json=data) as response:
        return await response.json()


async def main():
    async with aiohttp.ClientSession() as session:
        result = await post_data(
            session,
            "https://httpbin.org/post",
            {"name": "Иван", "age": 25}
        )
        print(result['json'])


asyncio.run(main())
3.5. Сессии и заголовки
python
import aiohttp
import asyncio


async def main():
    headers = {
        'User-Agent': 'MyApp/1.0',
        'Accept': 'application/json',
        'Authorization': 'Bearer token123'
    }
    
    async with aiohttp.ClientSession(headers=headers) as session:
        # Все запросы в сессии будут использовать эти заголовки
        async with session.get("https://httpbin.org/headers") as resp:
            data = await resp.json()
            print(data)


asyncio.run(main())
4. Асинхронные HTTP-запросы
4.1. Множественные запросы (параллельно)
python
import aiohttp
import asyncio
import time


async def fetch(session, url):
    async with session.get(url) as response:
        return await response.json()


async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results


async def main():
    urls = [
        "https://httpbin.org/get?id=1",
        "https://httpbin.org/get?id=2",
        "https://httpbin.org/get?id=3",
        "https://httpbin.org/get?id=4",
        "https://httpbin.org/get?id=5"
    ]
    
    start = time.time()
    results = await fetch_all(urls)
    elapsed = time.time() - start
    
    print(f"Получено {len(results)} результатов за {elapsed:.2f} сек")


asyncio.run(main())
4.2. Контроль параллелизма (Semaphore)
python
import aiohttp
import asyncio


async def fetch(session, url, semaphore):
    async with semaphore:  # Ограничиваем количество одновременных запросов
        async with session.get(url) as response:
            return await response.json()


async def fetch_all_with_limit(urls, limit=3):
    semaphore = asyncio.Semaphore(limit)
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url, semaphore) for url in urls]
        results = await asyncio.gather(*tasks)
        return results


async def main():
    urls = [f"https://httpbin.org/delay/1?id={i}" for i in range(10)]
    
    start = asyncio.get_event_loop().time()
    results = await fetch_all_with_limit(urls, limit=3)
    elapsed = asyncio.get_event_loop().time() - start
    
    print(f"Завершено за {elapsed:.2f} секунд")


asyncio.run(main())
4.3. Асинхронный клиент API
python
import aiohttp
import asyncio
from typing import List, Dict, Any


class AsyncAPIClient:
    """Асинхронный клиент для работы с API."""
    
    def __init__(self, base_url: str):
        self.base_url = base_url
        self.session = None
    
    async def __aenter__(self):
        self.session = aiohttp.ClientSession()
        return self
    
    async def __aexit__(self, *args):
        await self.session.close()
    
    async def get(self, endpoint: str, params: dict = None) -> Dict:
        async with self.session.get(
            f"{self.base_url}/{endpoint}", params=params
        ) as response:
            return await response.json()
    
    async def post(self, endpoint: str, data: dict) -> Dict:
        async with self.session.post(
            f"{self.base_url}/{endpoint}", json=data
        ) as response:
            return await response.json()
    
    async def get_all_users(self) -> List[Dict]:
        return await self.get("users")
    
    async def get_user(self, user_id: int) -> Dict:
        return await self.get(f"users/{user_id}")


async def main():
    async with AsyncAPIClient("https://jsonplaceholder.typicode.com") as client:
        users = await client.get_all_users()
        print(f"Получено {len(users)} пользователей")
        
        user = await client.get_user(1)
        print(f"Пользователь: {user['name']}")


asyncio.run(main())
5. Обработка ошибок и таймауты
5.1. Обработка исключений
python
import aiohttp
import asyncio


async def safe_fetch(url):
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(url, timeout=aiohttp.ClientTimeout(total=5)) as resp:
                return await resp.json()
    except aiohttp.ClientError as e:
        print(f"Клиентская ошибка: {e}")
        return None
    except asyncio.TimeoutError:
        print(f"Таймаут: {url}")
        return None
    except Exception as e:
        print(f"Неизвестная ошибка: {e}")
        return None


async def main():
    urls = [
        "https://httpbin.org/get",
        "https://httpbin.org/status/404",
        "https://nonexistent.url",
        "https://httpbin.org/delay/10"  # вызовет таймаут
    ]
    
    tasks = [safe_fetch(url) for url in urls]
    results = await asyncio.gather(*tasks)
    
    for url, result in zip(urls, results):
        print(f"{url}: {'OK' if result else 'FAILED'}")


asyncio.run(main())
5.2. Retry с экспоненциальной задержкой
python
import aiohttp
import asyncio


async def fetch_with_retry(
    url: str,
    max_retries: int = 3,
    base_delay: float = 1.0
):
    """Выполняет запрос с повторными попытками."""
    
    for attempt in range(1, max_retries + 1):
        try:
            async with aiohttp.ClientSession() as session:
                async with session.get(url, timeout=5) as response:
                    response.raise_for_status()
                    return await response.json()
                    
        except (aiohttp.ClientError, asyncio.TimeoutError) as e:
            if attempt == max_retries:
                print(f"Не удалось после {max_retries} попыток: {url}")
                return None
            
            delay = base_delay * (2 ** (attempt - 1))  # экспоненциальная задержка
            print(f"Попытка {attempt} не удалась. Повтор через {delay} сек...")
            await asyncio.sleep(delay)
    
    return None


async def main():
    result = await fetch_with_retry("https://httpbin.org/get")
    print(result)


asyncio.run(main())
5.3. Таймауты на уровне сессии
python
import aiohttp
import asyncio


async def main():
    # Глобальный таймаут для сессии
    timeout = aiohttp.ClientTimeout(total=10, connect=5)
    
    async with aiohttp.ClientSession(timeout=timeout) as session:
        try:
            async with session.get("https://httpbin.org/delay/15") as resp:
                data = await resp.json()
                print(data)
        except asyncio.TimeoutError:
            print("Превышен общий таймаут сессии")


asyncio.run(main())
6. Практические примеры
6.1. Асинхронный парсер сайтов
python
import aiohttp
import asyncio
from bs4 import BeautifulSoup
from typing import List, Dict


class AsyncWebScraper:
    """Асинхронный веб-скрапер."""
    
    def __init__(self, max_concurrent: int = 5):
        self.semaphore = asyncio.Semaphore(max_concurrent)
    
    async def fetch_page(self, session: aiohttp.ClientSession, url: str) -> str:
        async with self.semaphore:
            try:
                async with session.get(url, timeout=10) as response:
                    return await response.text()
            except Exception as e:
                print(f"Ошибка {url}: {e}")
                return ""
    
    async def parse_title(self, html: str) -> str:
        soup = BeautifulSoup(html, 'html.parser')
        title_tag = soup.find('title')
        return title_tag.text if title_tag else "Без названия"
    
    async def scrape_urls(self, urls: List[str]) -> List[Dict]:
        async with aiohttp.ClientSession() as session:
            tasks = [self.fetch_page(session, url) for url in urls]
            pages = await asyncio.gather(*tasks)
            
            results = []
            for url, html in zip(urls, pages):
                if html:
                    title = await self.parse_title(html)
                    results.append({"url": url, "title": title})
            
            return results


async def main():
    urls = [
        "https://www.python.org",
        "https://www.github.com",
        "https://www.stackoverflow.com",
        "https://www.reddit.com"
    ]
    
    scraper = AsyncWebScraper(max_concurrent=2)
    results = await scraper.scrape_urls(urls)
    
    for result in results:
        print(f"{result['url']} -> {result['title']}")


asyncio.run(main())
6.2. Асинхронный ETL с API
python
import aiohttp
import asyncio
import json
from datetime import datetime


class AsyncETLPipeline:
    """Асинхронный ETL-пайплайн для загрузки данных из API."""
    
    def __init__(self, api_url: str, output_file: str):
        self.api_url = api_url
        self.output_file = output_file
    
    async def extract_user(self, session: aiohttp.ClientSession, user_id: int) -> dict:
        """Извлечение данных одного пользователя."""
        async with session.get(f"{self.api_url}/users/{user_id}") as response:
            return await response.json()
    
    async def extract_all_users(self, user_ids: list) -> list:
        """Параллельное извлечение всех пользователей."""
        async with aiohttp.ClientSession() as session:
            tasks = [self.extract_user(session, uid) for uid in user_ids]
            users = await asyncio.gather(*tasks)
            return users
    
    def transform(self, users: list) -> list:
        """Трансформация данных."""
        transformed = []
        for user in users:
            transformed.append({
                "id": user.get('id'),
                "full_name": user.get('name'),
                "email": user.get('email'),
                "city": user.get('address', {}).get('city'),
                "processed_at": datetime.now().isoformat()
            })
        return transformed
    
    def load(self, data: list):
        """Загрузка в JSON файл."""
        with open(self.output_file, 'w', encoding='utf-8') as f:
            json.dump(data, f, indent=2, ensure_ascii=False)
        print(f"Сохранено {len(data)} записей")
    
    async def run(self, user_ids: list):
        """Запуск ETL-процесса."""
        print("Извлечение данных...")
        users = await self.extract_all_users(user_ids)
        
        print("Трансформация...")
        transformed = self.transform(users)
        
        print("Загрузка...")
        self.load(transformed)
        
        print("ETL завершён!")


async def main():
    api_url = "https://jsonplaceholder.typicode.com"
    user_ids = list(range(1, 11))
    
    etl = AsyncETLPipeline(api_url, "users_async.json")
    await etl.run(user_ids)


asyncio.run(main())
6.3. Полный пример: мониторинг API
python
import aiohttp
import asyncio
from datetime import datetime
from typing import Dict, List


class APIMonitor:
    """Асинхронный монитор API-эндпоинтов."""
    
    def __init__(self, endpoints: Dict[str, str], check_interval: int = 60):
        self.endpoints = endpoints  # {name: url}
        self.check_interval = check_interval
        self.results = []
    
    async def check_endpoint(self, session: aiohttp.ClientSession, name: str, url: str):
        """Проверка одного эндпоинта."""
        start = datetime.now()
        try:
            async with session.get(url, timeout=5) as response:
                status = response.status
                is_ok = 200 <= status < 300
        except Exception as e:
            status = 0
            is_ok = False
        
        end = datetime.now()
        response_time = (end - start).total_seconds()
        
        result = {
            "name": name,
            "url": url,
            "status": status,
            "is_ok": is_ok,
            "response_time": response_time,
            "timestamp": end.isoformat()
        }
        return result
    
    async def check_all(self):
        """Проверка всех эндпоинтов параллельно."""
        async with aiohttp.ClientSession() as session:
            tasks = [
                self.check_endpoint(session, name, url)
                for name, url in self.endpoints.items()
            ]
            results = await asyncio.gather(*tasks)
            self.results = results
            return results
    
    async def monitor(self, duration_seconds: int = 60):
        """Мониторинг в течение заданного времени."""
        start = datetime.now()
        checks_count = 0
        
        while (datetime.now() - start).total_seconds() < duration_seconds:
            print(f"\n🔍 Проверка #{checks_count + 1} в {datetime.now().strftime('%H:%M:%S')}")
            
            results = await self.check_all()
            for result in results:
                icon = "✅" if result["is_ok"] else "❌"
                print(f"  {icon} {result['name']}: {result['status']} ({result['response_time']:.2f}s)")
            
            checks_count += 1
            await asyncio.sleep(self.check_interval)
        
        return self.generate_report()
    
    def generate_report(self) -> Dict:
        """Генерация отчёта."""
        if not self.results:
            return {"error": "Нет данных"}
        
        total = len(self.results)
        successful = sum(1 for r in self.results if r["is_ok"])
        avg_time = sum(r["response_time"] for r in self.results) / total
        
        return {
            "total_endpoints": total,
            "successful": successful,
            "failed": total - successful,
            "availability": f"{successful / total * 100:.1f}%",
            "avg_response_time": f"{avg_time:.3f}s",
            "timestamp": datetime.now().isoformat()
        }


async def main():
    endpoints = {
        "GitHub API": "https://api.github.com/zen",
        "JSONPlaceholder": "https://jsonplaceholder.typicode.com/posts/1",
        "HttpBin": "https://httpbin.org/get",
        "Slow API": "https://httpbin.org/delay/2"
    }
    
    monitor = APIMonitor(endpoints, check_interval=10)
    
    print("Запуск мониторинга на 30 секунд...")
    report = await monitor.monitor(duration_seconds=30)
    
    print("\n" + "=" * 50)
    print("ИТОГОВЫЙ ОТЧЁТ")
    print("=" * 50)
    for key, value in report.items():
        print(f"{key}: {value}")


asyncio.run(main())
7. Контрольные вопросы
Что такое корутина в Python и как её объявить?

В чём разница между asyncio.gather() и asyncio.wait()?

Как ограничить количество одновременных запросов?

Как установить таймаут для асинхронного запроса?

Как отменить выполняющуюся задачу?

Как обработать ошибку в асинхронном коде?

Что делает async with?

Как реализовать повторные попытки при ошибке?

В чём преимущество асинхронности перед многопоточностью?

Когда НЕ стоит использовать асинхронность?

8. Практическое задание
Задание 1 (базовое)
Напишите асинхронную функцию, которая выполняет GET запросы к 5 разным URL и возвращает их статус-коды.

Задание 2 (среднее)
Создайте асинхронный клиент для погодного API, который параллельно получает данные о погоде для 10 городов.

Задание 3 (сложное)
Разработайте асинхронный ETL-пайплайн, который:

Загружает данные из нескольких API параллельно

Трансформирует и агрегирует данные

Загружает результат в JSON

Содержит обработку ошибок и повторные попытки

9. Шпаргалка
python
# === ОСНОВНЫЕ КОНСТРУКЦИИ ===
async def my_coro():
    await asyncio.sleep(1)
    return "result"

# === ЗАПУСК ===
asyncio.run(my_coro())

# === ПАРАЛЛЕЛЬНЫЙ ЗАПУСК ===
results = await asyncio.gather(coro1(), coro2(), coro3())

# === ТАЙМАУТ ===
result = await asyncio.wait_for(coro(), timeout=5)

# === Aiohttp ===
async with aiohttp.ClientSession() as session:
    async with session.get(url) as response:
        data = await response.json()

# === ОГРАНИЧЕНИЕ ПАРАЛЛЕЛИЗМА ===
semaphore = asyncio.Semaphore(5)
async with semaphore:
    await make_request()

# === ОБРАБОТКА ОШИБОК ===
try:
    await coro()
except asyncio.TimeoutError:
    pass

# === ОТМЕНА ЗАДАЧИ ===
task = asyncio.create_task(coro())
task.cancel()
Итог лекции
Вы сегодня:

✅ Изучили основы асинхронного программирования (async/await)

✅ Освоили библиотеку aiohttp для асинхронных HTTP-запросов

✅ Научились выполнять множество запросов параллельно

✅ Изучили контроль параллелизма и обработку ошибок

✅ Создали практические примеры: парсер, ETL, мониторинг

Теперь вы можете создавать высокопроизводительные асинхронные приложения!

