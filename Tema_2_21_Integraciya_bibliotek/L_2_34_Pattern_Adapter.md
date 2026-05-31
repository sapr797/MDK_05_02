# Тема 2.34. Паттерн Adapter для интеграции библиотек

**Цель лекции:**  
Изучить структурный паттерн Adapter и его применение для интеграции сторонних библиотек. Научиться создавать адаптеры, унифицирующие интерфейсы различных библиотек, и использовать их для упрощения замены зависимостей.

> Главная мысль: **Библиотеки меняются, а код вашего приложения — нет. Adapter — это прослойка, которая защищает вас от изменений внешних зависимостей.**

---

## Содержание

1. [Введение в паттерн Adapter](#1-введение-в-паттерн-adapter)
2. [Проблемы при интеграции библиотек](#2-проблемы-при-интеграции-библиотек)
3. [Реализация паттерна Adapter](#3-реализация-паттерна-adapter)
4. [Адаптация различных типов библиотек](#4-адаптация-различных-типов-библиотек)
5. [Тестирование адаптеров](#5-тестирование-адаптеров)
6. [Практические примеры](#6-практические-примеры)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Введение в паттерн Adapter

### 1.1. Что такое паттерн Adapter

**Adapter (Адаптер)** — структурный паттерн проектирования, который позволяет объектам с несовместимыми интерфейсами работать вместе.
┌─────────────────────────────────────────────────────────────────────────────┐
│ ПАТТЕРН ADAPTER │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ┌─────────────────┐ ┌─────────────────┐ │
│ │ Ваше приложение │ │ Сторонняя │ │
│ │ (ожидает │ │ библиотека │ │
│ │ интерфейс А) │ │ (интерфейс Б) │ │
│ └────────┬────────┘ └────────┬────────┘ │
│ │ │ │
│ │ │ │
│ ▼ ▼ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ ADAPTER │ │
│ │ • Получает вызовы в интерфейсе А │ │
│ │ • Преобразует в вызовы интерфейса Б │ │
│ │ • Адаптирует параметры и возвращаемые значения │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Когда нужен Adapter

| Ситуация | Пример | Решение |
|----------|--------|---------|
| **Смена библиотеки** | Замена `requests` на `aiohttp` | Адаптер для HTTP-клиента |
| **Разные версии** | Обновление библиотеки с breaking changes | Адаптер версии |
| **Унификация API** | Разные библиотеки для работы с БД | Единый интерфейс |
| **Тестирование** | Подмена реальной библиотеки моком | Адаптер-прокси |

### 1.3. Преимущества использования Adapter

| Преимущество | Описание |
|--------------|----------|
| **Изоляция** | Код приложения не зависит от конкретной библиотеки |
| **Заменяемость** | Можно легко заменить одну библиотеку на другую |
| **Тестируемость** | Легко подменять реальные библиотеки моками |
| **Единообразие** | Унифицированный интерфейс для разных библиотек |

---

## 2. Проблемы при интеграции библиотек

### 2.1. Несовместимость интерфейсов

```python
# Проблема: разные библиотеки имеют разные интерфейсы

# Библиотека A
class LibraryA:
    def send_request(self, url, method="GET"):
        # реализация
        pass

# Библиотека B
class LibraryB:
    def fetch(self, endpoint, http_method="GET"):
        # реализация
        pass

# Ваш код должен работать с обеими
def process_data(client):
    # Как вызвать, не зная тип клиента?
    pass
2.2. Зависимость от конкретной библиотеки
python
# Плохо: жёсткая привязка к конкретной библиотеке
import aiohttp

class DataFetcher:
    async def fetch(self, url):
        async with aiohttp.ClientSession() as session:
            async with session.get(url) as response:
                return await response.json()

# При смене библиотеки придётся переписывать весь код
2.3. Разные форматы данных
python
# Проблема: библиотеки возвращают данные в разных форматах
# Library A возвращает: {"status": 200, "body": {...}}
# Library B возвращает: {"code": 200, "data": {...}}
3. Реализация паттерна Adapter
3.1. Классическая реализация (через наследование)
python
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional


# ============================================================
# ЦЕЛЕВОЙ ИНТЕРФЕЙС (ожидаемый приложением)
# ============================================================

class HttpClient(ABC):
    """Абстрактный интерфейс HTTP-клиента."""
    
    @abstractmethod
    def get(self, url: str, params: Optional[Dict] = None) -> Dict:
        """Выполняет GET запрос."""
        pass
    
    @abstractmethod
    def post(self, url: str, data: Optional[Dict] = None) -> Dict:
        """Выполняет POST запрос."""
        pass


# ============================================================
# АДАПТИРУЕМЫЕ КЛАССЫ (сторонние библиотеки)
# ============================================================

class RequestsLibrary:
    """Библиотека requests (адаптируемая)."""
    
    def get_request(self, url: str, query_params: Optional[Dict] = None) -> Any:
        import requests
        response = requests.get(url, params=query_params)
        return response
    
    def post_request(self, url: str, body: Optional[Dict] = None) -> Any:
        import requests
        response = requests.post(url, json=body)
        return response


class AiohttpLibrary:
    """Библиотека aiohttp (адаптируемая)."""
    
    async def fetch(self, url: str, params: Optional[Dict] = None) -> Any:
        import aiohttp
        async with aiohttp.ClientSession() as session:
            async with session.get(url, params=params) as response:
                return await response.json()


# ============================================================
# АДАПТЕРЫ
# ============================================================

class RequestsAdapter(HttpClient):
    """Адаптер для библиотеки requests."""
    
    def __init__(self):
        self._library = RequestsLibrary()
    
    def get(self, url: str, params: Optional[Dict] = None) -> Dict:
        response = self._library.get_request(url, params)
        return {
            "status": response.status_code,
            "data": response.json() if response.text else None
        }
    
    def post(self, url: str, data: Optional[Dict] = None) -> Dict:
        response = self._library.post_request(url, data)
        return {
            "status": response.status_code,
            "data": response.json() if response.text else None
        }


class AiohttpAdapter(HttpClient):
    """Адаптер для библиотеки aiohttp."""
    
    def __init__(self):
        self._library = AiohttpLibrary()
    
    def get(self, url: str, params: Optional[Dict] = None) -> Dict:
        import asyncio
        # aiohttp асинхронный, поэтому нужен особый подход
        # В реальном приложении get должен быть async
        loop = asyncio.get_event_loop()
        result = loop.run_until_complete(self._library.fetch(url, params))
        return {
            "status": 200,
            "data": result
        }
    
    def post(self, url: str, data: Optional[Dict] = None) -> Dict:
        # аналогичная реализация
        return {"status": 200, "data": {}}


# ============================================================
# КЛИЕНТСКИЙ КОД (не зависит от конкретной библиотеки)
# ============================================================

class DataProcessor:
    """Обработчик данных, использующий абстракцию HttpClient."""
    
    def __init__(self, http_client: HttpClient):
        self.http_client = http_client
    
    def fetch_data(self, url: str) -> Dict:
        return self.http_client.get(url)


# Использование
processor = DataProcessor(RequestsAdapter())
data = processor.fetch_data("https://api.example.com/users")
3.2. Реализация через композицию
python
class HttpAdapter:
    """
    Адаптер через композицию — более гибкий подход.
    Позволяет динамически менять реализацию.
    """
    
    def __init__(self, library):
        self._library = library
    
    def get(self, url: str, **kwargs) -> Dict:
        # Проверяем тип библиотеки и вызываем соответствующий метод
        if hasattr(self._library, 'get_request'):
            response = self._library.get_request(url, kwargs.get('params'))
            return {"status": response.status_code, "data": response.json()}
        elif hasattr(self._library, 'fetch'):
            import asyncio
            result = asyncio.run(self._library.fetch(url, kwargs.get('params')))
            return {"status": 200, "data": result}
        else:
            raise ValueError(f"Unsupported library: {type(self._library)}")


# Использование
requests_adapter = HttpAdapter(RequestsLibrary())
aiohttp_adapter = HttpAdapter(AiohttpLibrary())
3.3. Фабрика адаптеров
python
class AdapterFactory:
    """Фабрика для создания адаптеров по типу библиотеки."""
    
    _adapters = {
        'requests': RequestsAdapter,
        'aiohttp': AiohttpAdapter,
    }
    
    @classmethod
    def create(cls, library_type: str, **kwargs) -> HttpClient:
        """Создаёт адаптер для указанной библиотеки."""
        adapter_class = cls._adapters.get(library_type)
        if not adapter_class:
            raise ValueError(f"Unknown library type: {library_type}")
        return adapter_class(**kwargs)
    
    @classmethod
    def register(cls, name: str, adapter_class):
        """Регистрирует новый адаптер."""
        cls._adapters[name] = adapter_class


# Использование
client = AdapterFactory.create('requests')
processor = DataProcessor(client)
4. Адаптация различных типов библиотек
4.1. Адаптер для HTTP-клиентов
python
from typing import Dict, Any, Optional
from abc import ABC, abstractmethod


class HttpClient(ABC):
    """Единый интерфейс для HTTP-клиентов."""
    
    @abstractmethod
    def get(self, url: str, params: Optional[Dict] = None) -> Dict:
        pass
    
    @abstractmethod
    def post(self, url: str, data: Optional[Dict] = None) -> Dict:
        pass


class RequestsAdapter(HttpClient):
    """Адаптер для библиотеки requests."""
    
    def __init__(self, timeout: int = 30):
        import requests
        self._requests = requests
        self.timeout = timeout
    
    def get(self, url: str, params: Optional[Dict] = None) -> Dict:
        response = self._requests.get(url, params=params, timeout=self.timeout)
        return self._parse_response(response)
    
    def post(self, url: str, data: Optional[Dict] = None) -> Dict:
        response = self._requests.post(url, json=data, timeout=self.timeout)
        return self._parse_response(response)
    
    def _parse_response(self, response) -> Dict:
        return {
            "status": response.status_code,
            "data": response.json() if response.text else None,
            "headers": dict(response.headers)
        }


class AiohttpAdapter(HttpClient):
    """Адаптер для библиотеки aiohttp."""
    
    def __init__(self, timeout: int = 30):
        self.timeout = timeout
    
    async def get_async(self, url: str, params: Optional[Dict] = None) -> Dict:
        import aiohttp
        async with aiohttp.ClientSession() as session:
            async with session.get(url, params=params, timeout=self.timeout) as resp:
                return {
                    "status": resp.status,
                    "data": await resp.json(),
                    "headers": dict(resp.headers)
                }
    
    def get(self, url: str, params: Optional[Dict] = None) -> Dict:
        import asyncio
        return asyncio.run(self.get_async(url, params))
    
    def post(self, url: str, data: Optional[Dict] = None) -> Dict:
        import asyncio
        return asyncio.run(self._post_async(url, data))
    
    async def _post_async(self, url: str, data: Optional[Dict] = None) -> Dict:
        import aiohttp
        async with aiohttp.ClientSession() as session:
            async with session.post(url, json=data, timeout=self.timeout) as resp:
                return {
                    "status": resp.status,
                    "data": await resp.json(),
                    "headers": dict(resp.headers)
                }


class HttpxAdapter(HttpClient):
    """Адаптер для библиотеки httpx."""
    
    def __init__(self, timeout: int = 30):
        import httpx
        self._httpx = httpx
        self.timeout = timeout
    
    def get(self, url: str, params: Optional[Dict] = None) -> Dict:
        with self._httpx.Client(timeout=self.timeout) as client:
            response = client.get(url, params=params)
            return self._parse_response(response)
    
    def post(self, url: str, data: Optional[Dict] = None) -> Dict:
        with self._httpx.Client(timeout=self.timeout) as client:
            response = client.post(url, json=data)
            return self._parse_response(response)
    
    def _parse_response(self, response) -> Dict:
        return {
            "status": response.status_code,
            "data": response.json() if response.text else None,
            "headers": dict(response.headers)
        }
4.2. Адаптер для логирования
python
from abc import ABC, abstractmethod
from typing import Any


class Logger(ABC):
    """Универсальный интерфейс для логгеров."""
    
    @abstractmethod
    def debug(self, message: str, **kwargs) -> None:
        pass
    
    @abstractmethod
    def info(self, message: str, **kwargs) -> None:
        pass
    
    @abstractmethod
    def warning(self, message: str, **kwargs) -> None:
        pass
    
    @abstractmethod
    def error(self, message: str, **kwargs) -> None:
        pass


class LoggingAdapter(Logger):
    """Адаптер для стандартного модуля logging."""
    
    def __init__(self, name: str = "app"):
        import logging
        self.logger = logging.getLogger(name)
    
    def debug(self, message: str, **kwargs) -> None:
        self.logger.debug(message, extra=kwargs)
    
    def info(self, message: str, **kwargs) -> None:
        self.logger.info(message, extra=kwargs)
    
    def warning(self, message: str, **kwargs) -> None:
        self.logger.warning(message, extra=kwargs)
    
    def error(self, message: str, **kwargs) -> None:
        self.logger.error(message, extra=kwargs)


class LoguruAdapter(Logger):
    """Адаптер для библиотеки loguru."""
    
    def __init__(self):
        from loguru import logger
        self.logger = logger
    
    def debug(self, message: str, **kwargs) -> None:
        self.logger.debug(message, **kwargs)
    
    def info(self, message: str, **kwargs) -> None:
        self.logger.info(message, **kwargs)
    
    def warning(self, message: str, **kwargs) -> None:
        self.logger.warning(message, **kwargs)
    
    def error(self, message: str, **kwargs) -> None:
        self.logger.error(message, **kwargs)


class StructlogAdapter(Logger):
    """Адаптер для библиотеки structlog."""
    
    def __init__(self):
        import structlog
        self.logger = structlog.get_logger()
    
    def debug(self, message: str, **kwargs) -> None:
        self.logger.debug(message, **kwargs)
    
    def info(self, message: str, **kwargs) -> None:
        self.logger.info(message, **kwargs)
    
    def warning(self, message: str, **kwargs) -> None:
        self.logger.warning(message, **kwargs)
    
    def error(self, message: str, **kwargs) -> None:
        self.logger.error(message, **kwargs)
4.3. Адаптер для работы с БД
python
from abc import ABC, abstractmethod
from typing import List, Dict, Any, Optional


class Database(ABC):
    """Универсальный интерфейс для работы с БД."""
    
    @abstractmethod
    def connect(self) -> None:
        pass
    
    @abstractmethod
    def disconnect(self) -> None:
        pass
    
    @abstractmethod
    def execute(self, query: str, params: Optional[Dict] = None) -> List[Dict]:
        pass


class SQLiteAdapter(Database):
    """Адаптер для SQLite."""
    
    def __init__(self, db_path: str = ":memory:"):
        import sqlite3
        self.db_path = db_path
        self.connection = None
    
    def connect(self) -> None:
        import sqlite3
        self.connection = sqlite3.connect(self.db_path)
        self.connection.row_factory = sqlite3.Row
    
    def disconnect(self) -> None:
        if self.connection:
            self.connection.close()
    
    def execute(self, query: str, params: Optional[Dict] = None) -> List[Dict]:
        cursor = self.connection.execute(query, params or {})
        if query.strip().upper().startswith('SELECT'):
            return [dict(row) for row in cursor.fetchall()]
        self.connection.commit()
        return []


class PostgreSQLAdapter(Database):
    """Адаптер для PostgreSQL (psycopg2)."""
    
    def __init__(self, host: str, port: int, database: str, user: str, password: str):
        self.config = {
            'host': host,
            'port': port,
            'database': database,
            'user': user,
            'password': password
        }
        self.connection = None
    
    def connect(self) -> None:
        import psycopg2
        import psycopg2.extras
        self.connection = psycopg2.connect(**self.config)
    
    def disconnect(self) -> None:
        if self.connection:
            self.connection.close()
    
    def execute(self, query: str, params: Optional[Dict] = None) -> List[Dict]:
        cursor = self.connection.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
        cursor.execute(query, params or {})
        if query.strip().upper().startswith('SELECT'):
            return [dict(row) for row in cursor.fetchall()]
        self.connection.commit()
        return []


class AsyncPGAdapter(Database):
    """Адаптер для асинхронного PostgreSQL (asyncpg)."""
    
    def __init__(self, host: str, port: int, database: str, user: str, password: str):
        self.config = {
            'host': host,
            'port': port,
            'database': database,
            'user': user,
            'password': password
        }
        self.pool = None
    
    async def connect_async(self) -> None:
        import asyncpg
        self.pool = await asyncpg.create_pool(**self.config)
    
    def connect(self) -> None:
        import asyncio
        asyncio.run(self.connect_async())
    
    def disconnect(self) -> None:
        if self.pool:
            import asyncio
            asyncio.run(self.pool.close())
    
    def execute(self, query: str, params: Optional[Dict] = None) -> List[Dict]:
        import asyncio
        return asyncio.run(self._execute_async(query, params))
    
    async def _execute_async(self, query: str, params: Optional[Dict] = None) -> List[Dict]:
        async with self.pool.acquire() as conn:
            if query.strip().upper().startswith('SELECT'):
                rows = await conn.fetch(query, *(params.values() if params else []))
                return [dict(row) for row in rows]
            else:
                await conn.execute(query, *(params.values() if params else []))
                return []
5. Тестирование адаптеров
5.1. Mock-адаптер для тестирования
python
class MockHttpClient(HttpClient):
    """Mock-адаптер для тестирования."""
    
    def __init__(self):
        self.requests = []
        self.responses = {}
    
    def add_response(self, url: str, response: Dict, method: str = "GET"):
        self.responses[f"{method}:{url}"] = response
    
    def get(self, url: str, params: Optional[Dict] = None) -> Dict:
        self.requests.append(("GET", url, params))
        key = f"GET:{url}"
        return self.responses.get(key, {"status": 404, "data": None})
    
    def post(self, url: str, data: Optional[Dict] = None) -> Dict:
        self.requests.append(("POST", url, data))
        key = f"POST:{url}"
        return self.responses.get(key, {"status": 404, "data": None})


# Тестирование с Mock-адаптером
def test_data_processor():
    mock = MockHttpClient()
    mock.add_response("https://api.example.com/users", {
        "status": 200,
        "data": [{"id": 1, "name": "Ivan"}]
    })
    
    processor = DataProcessor(mock)
    result = processor.fetch_data("https://api.example.com/users")
    
    assert result["status"] == 200
    assert len(result["data"]) == 1
    assert len(mock.requests) == 1
5.2. Тестирование с реальным адаптером
python
import pytest
from unittest.mock import patch, MagicMock


class TestRequestsAdapter:
    """Тесты для адаптера requests."""
    
    @patch('requests.get')
    def test_get_success(self, mock_get):
        # Настройка мока
        mock_response = MagicMock()
        mock_response.status_code = 200
        mock_response.json.return_value = {"id": 1, "name": "Ivan"}
        mock_get.return_value = mock_response
        
        # Тестирование
        adapter = RequestsAdapter()
        result = adapter.get("https://api.example.com/users/1")
        
        # Проверки
        assert result["status"] == 200
        assert result["data"]["id"] == 1
        mock_get.assert_called_once()
    
    @patch('requests.get')
    def test_get_error(self, mock_get):
        mock_response = MagicMock()
        mock_response.status_code = 404
        mock_get.return_value = mock_response
        
        adapter = RequestsAdapter()
        result = adapter.get("https://api.example.com/users/999")
        
        assert result["status"] == 404
6. Практические примеры
6.1. Адаптер для работы с разными API погоды
python
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional


class WeatherService(ABC):
    """Абстрактный интерфейс для сервисов погоды."""
    
    @abstractmethod
    def get_current_weather(self, city: str) -> Dict:
        pass
    
    @abstractmethod
    def get_forecast(self, city: str, days: int = 5) -> Dict:
        pass


class OpenWeatherMapAdapter(WeatherService):
    """Адаптер для OpenWeatherMap API."""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = "https://api.openweathermap.org/data/2.5"
    
    def get_current_weather(self, city: str) -> Dict:
        import requests
        url = f"{self.base_url}/weather"
        params = {
            "q": city,
            "appid": self.api_key,
            "units": "metric"
        }
        response = requests.get(url, params=params)
        data = response.json()
        
        return {
            "temperature": data.get("main", {}).get("temp"),
            "humidity": data.get("main", {}).get("humidity"),
            "description": data.get("weather", [{}])[0].get("description"),
            "wind_speed": data.get("wind", {}).get("speed")
        }
    
    def get_forecast(self, city: str, days: int = 5) -> Dict:
        import requests
        url = f"{self.base_url}/forecast"
        params = {
            "q": city,
            "appid": self.api_key,
            "units": "metric",
            "cnt": days * 8
        }
        response = requests.get(url, params=params)
        data = response.json()
        
        forecast = []
        for item in data.get("list", []):
            forecast.append({
                "datetime": item.get("dt_txt"),
                "temperature": item.get("main", {}).get("temp"),
                "description": item.get("weather", [{}])[0].get("description")
            })
        
        return {"city": city, "forecast": forecast}


class WeatherAPIClient:
    """Клиент, использующий адаптер."""
    
    def __init__(self, weather_service: WeatherService):
        self.service = weather_service
    
    def get_weather_report(self, city: str) -> str:
        weather = self.service.get_current_weather(city)
        return (f"Погода в {city}: {weather['temperature']}°C, "
                f"{weather['description']}. Влажность: {weather['humidity']}%")


# Использование
weather_service = OpenWeatherMapAdapter(api_key="your-key")
client = WeatherAPIClient(weather_service)
print(client.get_weather_report("Moscow"))
6.2. Адаптер для разных версий библиотеки
python
# Ситуация: обновление библиотеки с breaking changes
# Старая версия: response.data
# Новая версия: response.body


class LibraryAdapter:
    """Адаптер для разных версий библиотеки."""
    
    @staticmethod
    def get_response_data(response) -> Dict:
        """Унифицированный доступ к данным ответа."""
        if hasattr(response, 'data'):
            return response.data
        elif hasattr(response, 'body'):
            return response.body
        else:
            raise AttributeError("Unknown response format")


# Использование
response = some_library.fetch(url)
data = LibraryAdapter.get_response_data(response)
7. Контрольные вопросы
Что такое паттерн Adapter и для чего он нужен?

В чём разница между адаптером через наследование и через композицию?

Как Adapter помогает при смене сторонней библиотеки?

Как протестировать код, использующий адаптер?

Что такое фабрика адаптеров и когда она нужна?

Какие проблемы решает паттерн Adapter при интеграции библиотек?

Как адаптировать асинхронную библиотеку к синхронному интерфейсу?

В чём отличие Adapter от Facade?

Как реализовать адаптер для логирования разных библиотек?

Как Adapter помогает в тестировании?

8. Практическое задание
Задание 1 (базовое)
Создайте адаптер для библиотеки requests, унифицирующий методы get, post, put, delete. Реализуйте обработку ошибок.

Задание 2 (среднее)
Создайте адаптер для работы с разными форматами сериализации (JSON, XML, YAML). Адаптер должен предоставлять единые методы serialize и deserialize.

Задание 3 (сложное)
Реализуйте адаптер для интеграции с разными базами данных (SQLite, PostgreSQL). Создайте унифицированный интерфейс для CRUD операций.

9. Шпаргалка
python
# === БАЗОВЫЙ ШАБЛОН ADAPTER ===
from abc import ABC, abstractmethod

class Target(ABC):
    @abstractmethod
    def request(self):
        pass

class Adaptee:
    def specific_request(self):
        return "specific"

class Adapter(Target):
    def __init__(self, adaptee: Adaptee):
        self._adaptee = adaptee
    
    def request(self):
        return self._adaptee.specific_request()

# === ФАБРИКА АДАПТЕРОВ ===
class AdapterFactory:
    _adapters = {}
    
    @classmethod
    def register(cls, name, adapter_class):
        cls._adapters[name] = adapter_class
    
    @classmethod
    def create(cls, name, **kwargs):
        return cls._adapters[name](**kwargs)

# === MOCK-АДАПТЕР ДЛЯ ТЕСТОВ ===
class MockAdapter(Target):
    def __init__(self):
        self.calls = []
        self.responses = {}
    
    def request(self):
        self.calls.append(("request",))
        return self.responses.get("request", {})
Итог лекции
Вы сегодня:

✅ Изучили паттерн Adapter и его применение

✅ Научились создавать адаптеры для сторонних библиотек

✅ Освоили унификацию интерфейсов различных библиотек

✅ Изучили тестирование кода с адаптерами

✅ Создали практические примеры адаптеров

Теперь вы можете интегрировать любые сторонние библиотеки, не завязывая на них свой код!

