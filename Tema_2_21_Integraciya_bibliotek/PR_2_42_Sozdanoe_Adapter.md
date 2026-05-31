# ПЗ 2.42. Создание адаптера для внешней библиотеки

**Тема:** Паттерн Adapter, интеграция библиотек, унификация интерфейсов

**Цель работы:**  
Научиться создавать адаптеры для внешних библиотек, унифицировать интерфейсы различных библиотек, обеспечивать возможность замены одной библиотеки на другую без изменения основного кода.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `requests`, `aiohttp`, `httpx`

```bash
pip install requests aiohttp httpx
Главная мысль: Адаптер — это прослойка, которая защищает ваш код от изменений внешних библиотек. С ним вы можете легко заменить одну библиотеку на другую.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Зачем нужен адаптер для библиотеки
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ПРОБЛЕМА БЕЗ АДАПТЕРА                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────┐         ┌─────────────────┐                         │
│   │   Ваше приложение │────────►│   Библиотека A  │                         │
│   │                 │         │   (requests)    │                         │
│   └─────────────────┘         └─────────────────┘                         │
│                                                                             │
│   Проблема: приложение жёстко привязано к библиотеке A                      │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                    РЕШЕНИЕ С АДАПТЕРОМ                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│   │   Ваше приложение │────────►│    Адаптер     │────────►│   Библиотека A  │
│   │                 │         │                 │         │   (requests)    │
│   └─────────────────┘         └─────────────────┘         └─────────────────┘
│          │                            │                                    │
│          │                            │         ┌─────────────────┐         │
│          │                            └────────►│   Библиотека B  │         │
│          │                                      │   (aiohttp)     │         │
│          │                                      └─────────────────┘         │
│          │                                                                  │
│          └──────────────────────────────────────────────────────────────────┘
│                                                                             │
│   Преимущества: приложение не зависит от конкретной библиотеки              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
1.2. Структура адаптера
Компонент	Описание
Target (Целевой интерфейс)	Интерфейс, ожидаемый приложением
Adaptee (Адаптируемый)	Внешняя библиотека с несовместимым интерфейсом
Adapter (Адаптер)	Класс, преобразующий интерфейс Adaptee в Target
Client (Клиент)	Приложение, использующее Target
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая создание адаптера для внешней библиотеки.

Техническое задание (нулевой вариант)
Создайте адаптер для HTTP-клиентов, который унифицирует интерфейсы трёх библиотек: requests, aiohttp, httpx. Адаптер должен предоставлять единые методы get и post, возвращающие данные в стандартизированном формате.

Эталонная реализация
python
#!/usr/bin/env python3
"""
http_adapter.py — Адаптер для HTTP-библиотек.
"""

from abc import ABC, abstractmethod
from typing import Dict, Any, Optional, Union
from dataclasses import dataclass
import asyncio


# ============================================================
# ЦЕЛЕВОЙ ИНТЕРФЕЙС (единый формат ответа)
# ============================================================

@dataclass
class HttpResponse:
    """Стандартизированный HTTP-ответ."""
    status_code: int
    data: Optional[Union[Dict, str]] = None
    headers: Dict[str, str] = None
    error: Optional[str] = None
    
    @property
    def is_success(self) -> bool:
        return 200 <= self.status_code < 300


class HttpClient(ABC):
    """Абстрактный интерфейс HTTP-клиента."""
    
    @abstractmethod
    def get(self, url: str, params: Optional[Dict] = None, headers: Optional[Dict] = None) -> HttpResponse:
        """Выполняет GET запрос."""
        pass
    
    @abstractmethod
    def post(self, url: str, data: Optional[Dict] = None, headers: Optional[Dict] = None) -> HttpResponse:
        """Выполняет POST запрос."""
        pass
    
    @abstractmethod
    def close(self) -> None:
        """Закрывает соединение (если нужно)."""
        pass


# ============================================================
# АДАПТЕР ДЛЯ БИБЛИОТЕКИ REQUESTS
# ============================================================

class RequestsAdapter(HttpClient):
    """Адаптер для библиотеки requests."""
    
    def __init__(self, timeout: int = 30):
        import requests
        self._requests = requests
        self.timeout = timeout
        self._session = None
    
    def _get_session(self):
        if self._session is None:
            self._session = self._requests.Session()
        return self._session
    
    def get(self, url: str, params: Optional[Dict] = None, headers: Optional[Dict] = None) -> HttpResponse:
        try:
            session = self._get_session()
            response = session.get(
                url,
                params=params,
                headers=headers or {},
                timeout=self.timeout
            )
            return HttpResponse(
                status_code=response.status_code,
                data=self._parse_response(response),
                headers=dict(response.headers)
            )
        except Exception as e:
            return HttpResponse(
                status_code=0,
                error=str(e)
            )
    
    def post(self, url: str, data: Optional[Dict] = None, headers: Optional[Dict] = None) -> HttpResponse:
        try:
            session = self._get_session()
            response = session.post(
                url,
                json=data,
                headers=headers or {},
                timeout=self.timeout
            )
            return HttpResponse(
                status_code=response.status_code,
                data=self._parse_response(response),
                headers=dict(response.headers)
            )
        except Exception as e:
            return HttpResponse(
                status_code=0,
                error=str(e)
            )
    
    def _parse_response(self, response):
        try:
            return response.json()
        except:
            return response.text
    
    def close(self) -> None:
        if self._session:
            self._session.close()
            self._session = None


# ============================================================
# АДАПТЕР ДЛЯ БИБЛИОТЕКИ AIOHTTP
# ============================================================

class AiohttpAdapter(HttpClient):
    """Адаптер для библиотеки aiohttp (синхронная обёртка)."""
    
    def __init__(self, timeout: int = 30):
        self.timeout = timeout
        self._loop = None
    
    def _get_loop(self):
        if self._loop is None:
            self._loop = asyncio.new_event_loop()
            asyncio.set_event_loop(self._loop)
        return self._loop
    
    def _run_async(self, coro):
        loop = self._get_loop()
        return loop.run_until_complete(coro)
    
    async def _get_async(self, url: str, params: Optional[Dict], headers: Optional[Dict]) -> HttpResponse:
        import aiohttp
        try:
            async with aiohttp.ClientSession() as session:
                async with session.get(
                    url,
                    params=params,
                    headers=headers or {},
                    timeout=aiohttp.ClientTimeout(total=self.timeout)
                ) as response:
                    try:
                        data = await response.json()
                    except:
                        data = await response.text()
                    return HttpResponse(
                        status_code=response.status,
                        data=data,
                        headers=dict(response.headers)
                    )
        except Exception as e:
            return HttpResponse(status_code=0, error=str(e))
    
    async def _post_async(self, url: str, data: Optional[Dict], headers: Optional[Dict]) -> HttpResponse:
        import aiohttp
        try:
            async with aiohttp.ClientSession() as session:
                async with session.post(
                    url,
                    json=data,
                    headers=headers or {},
                    timeout=aiohttp.ClientTimeout(total=self.timeout)
                ) as response:
                    try:
                        response_data = await response.json()
                    except:
                        response_data = await response.text()
                    return HttpResponse(
                        status_code=response.status,
                        data=response_data,
                        headers=dict(response.headers)
                    )
        except Exception as e:
            return HttpResponse(status_code=0, error=str(e))
    
    def get(self, url: str, params: Optional[Dict] = None, headers: Optional[Dict] = None) -> HttpResponse:
        return self._run_async(self._get_async(url, params, headers))
    
    def post(self, url: str, data: Optional[Dict] = None, headers: Optional[Dict] = None) -> HttpResponse:
        return self._run_async(self._post_async(url, data, headers))
    
    def close(self) -> None:
        if self._loop:
            self._loop.close()
            self._loop = None


# ============================================================
# АДАПТЕР ДЛЯ БИБЛИОТЕКИ HTTPX
# ============================================================

class HttpxAdapter(HttpClient):
    """Адаптер для библиотеки httpx."""
    
    def __init__(self, timeout: int = 30):
        import httpx
        self._httpx = httpx
        self.timeout = timeout
        self._client = None
    
    def _get_client(self):
        if self._client is None:
            self._client = self._httpx.Client(timeout=self.timeout)
        return self._client
    
    def get(self, url: str, params: Optional[Dict] = None, headers: Optional[Dict] = None) -> HttpResponse:
        try:
            client = self._get_client()
            response = client.get(url, params=params, headers=headers or {})
            return HttpResponse(
                status_code=response.status_code,
                data=self._parse_response(response),
                headers=dict(response.headers)
            )
        except Exception as e:
            return HttpResponse(status_code=0, error=str(e))
    
    def post(self, url: str, data: Optional[Dict] = None, headers: Optional[Dict] = None) -> HttpResponse:
        try:
            client = self._get_client()
            response = client.post(url, json=data, headers=headers or {})
            return HttpResponse(
                status_code=response.status_code,
                data=self._parse_response(response),
                headers=dict(response.headers)
            )
        except Exception as e:
            return HttpResponse(status_code=0, error=str(e))
    
    def _parse_response(self, response):
        try:
            return response.json()
        except:
            return response.text
    
    def close(self) -> None:
        if self._client:
            self._client.close()
            self._client = None


# ============================================================
# ФАБРИКА АДАПТЕРОВ
# ============================================================

class HttpClientFactory:
    """Фабрика для создания адаптеров HTTP-клиентов."""
    
    _adapters = {
        'requests': RequestsAdapter,
        'aiohttp': AiohttpAdapter,
        'httpx': HttpxAdapter,
    }
    
    @classmethod
    def create(cls, library: str, **kwargs) -> HttpClient:
        """
        Создаёт адаптер для указанной библиотеки.
        
        Args:
            library: Название библиотеки ('requests', 'aiohttp', 'httpx')
            **kwargs: Дополнительные параметры (timeout и т.д.)
        
        Returns:
            HttpClient: Адаптер для указанной библиотеки
        """
        adapter_class = cls._adapters.get(library.lower())
        if not adapter_class:
            raise ValueError(f"Unknown library: {library}. Available: {list(cls._adapters.keys())}")
        return adapter_class(**kwargs)
    
    @classmethod
    def register(cls, name: str, adapter_class):
        """Регистрирует новый адаптер."""
        cls._adapters[name] = adapter_class


# ============================================================
# КЛИЕНТСКИЙ КОД (не зависит от конкретной библиотеки)
# ============================================================

class DataFetcher:
    """
    Класс для получения данных, использующий HttpClient.
    Не зависит от конкретной HTTP-библиотеки.
    """
    
    def __init__(self, http_client: HttpClient):
        self.http_client = http_client
    
    def fetch_user(self, user_id: int) -> Optional[Dict]:
        """Получает данные пользователя."""
        url = f"https://jsonplaceholder.typicode.com/users/{user_id}"
        response = self.http_client.get(url)
        
        if response.is_success:
            return response.data
        else:
            print(f"Ошибка: {response.status_code} - {response.error}")
            return None
    
    def fetch_users(self, limit: int = 5) -> List[Dict]:
        """Получает список пользователей."""
        url = "https://jsonplaceholder.typicode.com/users"
        response = self.http_client.get(url, params={'_limit': limit})
        
        if response.is_success:
            return response.data
        return []
    
    def create_post(self, title: str, body: str, user_id: int) -> Optional[Dict]:
        """Создаёт новый пост."""
        url = "https://jsonplaceholder.typicode.com/posts"
        data = {
            'title': title,
            'body': body,
            'userId': user_id
        }
        response = self.http_client.post(url, data=data)
        
        if response.is_success:
            return response.data
        return None


# ============================================================
# ТЕСТИРОВАНИЕ АДАПТЕРОВ
# ============================================================

class MockHttpClient(HttpClient):
    """Mock-адаптер для тестирования."""
    
    def __init__(self):
        self._responses = {}
        self.requests_log = []
    
    def add_response(self, url: str, response: HttpResponse, method: str = "GET"):
        self._responses[f"{method}:{url}"] = response
    
    def get(self, url: str, params: Optional[Dict] = None, headers: Optional[Dict] = None) -> HttpResponse:
        self.requests_log.append(("GET", url, params))
        key = f"GET:{url}"
        return self._responses.get(key, HttpResponse(status_code=404))
    
    def post(self, url: str, data: Optional[Dict] = None, headers: Optional[Dict] = None) -> HttpResponse:
        self.requests_log.append(("POST", url, data))
        key = f"POST:{url}"
        return self._responses.get(key, HttpResponse(status_code=404))
    
    def close(self) -> None:
        pass


# ============================================================
# ДЕМОНСТРАЦИЯ
# ============================================================

def demo_with_mock():
    """Демонстрация с mock-адаптером (без реальных запросов)."""
    print("\n" + "=" * 60)
    print("ДЕМОНСТРАЦИЯ С MOCK-АДАПТЕРОМ")
    print("=" * 60)
    
    # Создаём mock-клиент
    mock = MockHttpClient()
    mock.add_response(
        "https://jsonplaceholder.typicode.com/users/1",
        HttpResponse(status_code=200, data={"id": 1, "name": "Mock User"})
    )
    
    # Используем DataFetcher с mock-клиентом
    fetcher = DataFetcher(mock)
    user = fetcher.fetch_user(1)
    
    print(f"Результат: {user}")
    print(f"Выполненные запросы: {mock.requests_log}")


def demo_with_real_library(library: str = "requests"):
    """Демонстрация с реальной библиотекой."""
    print("\n" + "=" * 60)
    print(f"ДЕМОНСТРАЦИЯ С БИБЛИОТЕКОЙ {library.upper()}")
    print("=" * 60)
    
    try:
        # Создаём адаптер через фабрику
        client = HttpClientFactory.create(library, timeout=10)
        fetcher = DataFetcher(client)
        
        # Получение пользователя
        print("\n📥 Получение пользователя #1...")
        user = fetcher.fetch_user(1)
        if user:
            print(f"   Имя: {user.get('name')}")
            print(f"   Email: {user.get('email')}")
        
        # Получение списка пользователей
        print("\n📋 Получение списка пользователей...")
        users = fetcher.fetch_users(limit=3)
        print(f"   Получено пользователей: {len(users)}")
        for u in users[:3]:
            print(f"   • {u.get('name')}")
        
        # Создание поста
        print("\n📝 Создание нового поста...")
        post = fetcher.create_post(
            title="Тестовый пост",
            body="Содержание тестового поста",
            user_id=1
        )
        if post:
            print(f"   Пост создан! ID: {post.get('id')}")
        
        client.close()
        
    except ImportError as e:
        print(f"\n❌ Библиотека {library} не установлена")
        print(f"   Установите: pip install {library}")
    except Exception as e:
        print(f"\n❌ Ошибка: {e}")


def demo_compare_libraries():
    """Сравнение работы разных библиотек через адаптер."""
    print("\n" + "=" * 60)
    print("СРАВНЕНИЕ РАБОТЫ БИБЛИОТЕК")
    print("=" * 60)
    
    libraries = ['requests', 'aiohttp', 'httpx']
    
    for library in libraries:
        print(f"\n📚 Тестирование {library}...")
        try:
            client = HttpClientFactory.create(library, timeout=5)
            fetcher = DataFetcher(client)
            
            user = fetcher.fetch_user(1)
            if user:
                print(f"   ✅ {library} работает корректно")
                print(f"   Имя пользователя: {user.get('name')}")
            else:
                print(f"   ⚠️ {library} вернул пустой ответ")
            
            client.close()
            
        except ImportError:
            print(f"   ❌ {library} не установлена")
        except Exception as e:
            print(f"   ❌ Ошибка: {e}")


def main():
    """Главная функция."""
    print("=" * 70)
    print("СОЗДАНИЕ АДАПТЕРА ДЛЯ HTTP-БИБЛИОТЕК")
    print("=" * 70)
    
    # Демонстрация с mock
    demo_with_mock()
    
    # Демонстрация с реальными библиотеками
    demo_with_real_library("requests")
    
    # Сравнение библиотек
    demo_compare_libraries()
    
    print("\n" + "=" * 70)
    print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
    print("=" * 70)


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
======================================================================
СОЗДАНИЕ АДАПТЕРА ДЛЯ HTTP-БИБЛИОТЕК
======================================================================

============================================================
ДЕМОНСТРАЦИЯ С MOCK-АДАПТЕРОМ
============================================================
Результат: {'id': 1, 'name': 'Mock User'}
Выполненные запросы: [('GET', 'https://jsonplaceholder.typicode.com/users/1', None)]

============================================================
ДЕМОНСТРАЦИЯ С БИБЛИОТЕКОЙ REQUESTS
============================================================

📥 Получение пользователя #1...
   Имя: Leanne Graham
   Email: Sincere@april.biz

📋 Получение списка пользователей...
   Получено пользователей: 3
   • Leanne Graham
   • Ervin Howell
   • Clementine Bauch

📝 Создание нового поста...
   Пост создан! ID: 101

============================================================
СРАВНЕНИЕ РАБОТЫ БИБЛИОТЕК
============================================================

📚 Тестирование requests...
   ✅ requests работает корректно
   Имя пользователя: Leanne Graham

📚 Тестирование aiohttp...
   ✅ aiohttp работает корректно
   Имя пользователя: Leanne Graham

📚 Тестирование httpx...
   ✅ httpx работает корректно
   Имя пользователя: Leanne Graham

======================================================================
ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА
======================================================================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (один адаптер)

Варианты 9-17: средний (несколько адаптеров + фабрика)

Варианты 18-25: сложный (асинхронные адаптеры + тесты)

Варианты 1-8 (Базовый уровень)
№	Библиотека	Целевой интерфейс
1	requests	HttpClient
2	aiohttp	AsyncHttpClient
3	httpx	HttpClient
4	python-dateutil	DateParser
5	pytz	TimezoneConverter
6	Pillow	ImageProcessor
7	loguru	Logger
8	redis	CacheClient
Варианты 9-17 (Средний уровень)
№	Библиотеки	Адаптер	Фабрика
9	requests, aiohttp	HttpAdapter	✅
10	python-dateutil, arrow	DateAdapter	✅
11	loguru, structlog	LogAdapter	✅
12	redis, memcached	CacheAdapter	✅
13	sqlite3, psycopg2	DbAdapter	✅
14	beautifulsoup4, lxml	ParserAdapter	✅
15	yagmail, smtplib	MailAdapter	✅
16	pytest, unittest	TestAdapter	✅
17	click, argparse	CliAdapter	✅
Варианты 18-25 (Сложный уровень)
№	Библиотеки	Особенности
18	requests, aiohttp, httpx	+ тестирование, mock
19	asyncpg, aiomysql, aiosqlite	Асинхронные адаптеры
20	celery, rq, dramatiq	Очереди задач
21	prometheus_client, statsd	Метрики
22	pytest, nose, unittest	Тестовые адаптеры
23	fastapi, flask, aiohttp	Web-адаптеры
24	tensorflow, pytorch, sklearn	ML-адаптеры
25	kafka, rabbitmq, redis	Message brokers
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Адаптер не работает
3 (удовлетворительно)	Создан адаптер для одной библиотеки
4 (хорошо)	+ фабрика адаптеров
5 (отлично)	+ тестирование, обработка ошибок, документация
5. Шпаргалка
python
# === БАЗОВЫЙ ШАБЛОН АДАПТЕРА ===
from abc import ABC, abstractmethod

class Target(ABC):
    @abstractmethod
    def request(self):
        pass

class Adaptee:
    def specific_request(self):
        return "result"

class Adapter(Target):
    def __init__(self, adaptee: Adaptee):
        self._adaptee = adaptee
    
    def request(self):
        return self._adaptee.specific_request()

# === ФАБРИКА АДАПТЕРОВ ===
class AdapterFactory:
    _adapters = {}
    
    @classmethod
    def create(cls, name, **kwargs):
        return cls._adapters[name](**kwargs)

# === MOCK-АДАПТЕР ===
class MockAdapter(Target):
    def __init__(self):
        self.responses = {}
    
    def request(self):
        return self.responses.get('request', {})
Карточка студента
text
ПЗ 2.41. СОЗДАНИЕ АДАПТЕРА ДЛЯ ВНЕШНЕЙ БИБЛИОТЕКИ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== АДАПТИРУЕМЫЕ БИБЛИОТЕКИ ===

1. _____________
2. _____________
3. _____________

=== ЦЕЛЕВОЙ ИНТЕРФЕЙС ===

Название: _____________
Методы:
• _____________
• _____________
• _____________

=== КОМПОНЕНТЫ ===

□ Абстрактный класс Target
□ Адаптеры для каждой библиотеки
□ Фабрика адаптеров
□ Mock-адаптер для тестов
□ Клиентский код

=== ОТЧЁТ ===

Файл: adapter.py
Количество адаптеров: _____
Тесты: _____

Дата выполнения: _____________
