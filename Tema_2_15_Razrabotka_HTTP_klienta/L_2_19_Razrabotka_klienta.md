# Тема 2.19. Библиотека requests. GET, POST, PUT, DELETE запросы

**Цель лекции:**  
Изучить библиотеку `requests` для выполнения HTTP-запросов в Python, освоить методы GET, POST, PUT, DELETE, научиться работать с параметрами запросов, заголовками, телом запроса, обрабатывать ответы сервера.

> Главная мысль: **Библиотека requests — это "HTTP для людей". Простой, интуитивный интерфейс для работы с любыми веб-API.**

---

## Содержание

1. [Введение в библиотеку requests](#1-введение-в-библиотеку-requests)
2. [GET запросы](#2-get-запросы)
3. [POST запросы](#3-post-запросы)
4. [PUT и DELETE запросы](#4-put-и-delete-запросы)
5. [Обработка ответов](#5-обработка-ответов)
6. [Практические примеры](#6-практические-примеры)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Введение в библиотеку requests

### 1.1. Установка и импорт

```bash
# Установка библиотеки
pip install requests
python
# Импорт
import requests

# Проверка версии
print(requests.__version__)  # 2.31.0
1.2. Основные методы
Метод	Назначение	Пример
requests.get()	GET запрос	requests.get(url)
requests.post()	POST запрос	requests.post(url, json=data)
requests.put()	PUT запрос	requests.put(url, json=data)
requests.delete()	DELETE запрос	requests.delete(url)
requests.patch()	PATCH запрос	requests.patch(url, json=data)
requests.head()	HEAD запрос	requests.head(url)
requests.options()	OPTIONS запрос	requests.options(url)
1.3. Структура запроса и ответа
python
# Запрос
response = requests.get('https://api.example.com/users')

# Ответ
print(response.status_code)   # 200
print(response.headers)       # Заголовки ответа
print(response.text)          # Тело ответа (строка)
print(response.json())        # Тело ответа (парсинг JSON)
2. GET запросы
2.1. Базовый GET запрос
python
import requests

# Простой GET запрос
response = requests.get('https://jsonplaceholder.typicode.com/posts')

print(f"Статус: {response.status_code}")
print(f"Количество записей: {len(response.json())}")
2.2. GET с параметрами (query-параметры)
python
import requests

# GET с query-параметрами (способ 1: через params)
params = {
    'userId': 1,
    '_limit': 5
}

response = requests.get(
    'https://jsonplaceholder.typicode.com/posts',
    params=params
)

print(f"URL: {response.url}")  # https://.../posts?userId=1&_limit=5
print(response.json())

# GET с query-параметрами (способ 2: прямо в URL)
response = requests.get('https://jsonplaceholder.typicode.com/posts?userId=1&_limit=5')
2.3. GET с заголовками
python
import requests

# Добавление заголовков
headers = {
    'User-Agent': 'MyApp/1.0',
    'Accept': 'application/json',
    'Authorization': 'Bearer my_token_123'
}

response = requests.get(
    'https://api.github.com/user',
    headers=headers
)

print(response.status_code)
2.4. GET с таймаутом
python
import requests

try:
    # Таймаут в секундах
    response = requests.get('https://httpbin.org/delay/5', timeout=3)
    print(response.json())
except requests.exceptions.Timeout:
    print("Превышено время ожидания")
2.5. Пример: получение данных о пользователях
python
import requests

def get_users():
    """Получение списка пользователей."""
    url = 'https://jsonplaceholder.typicode.com/users'
    
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()  # Вызывает исключение при 4xx/5xx
        
        users = response.json()
        
        for user in users:
            print(f"{user['id']}: {user['name']} ({user['email']})")
        
        return users
        
    except requests.exceptions.RequestException as e:
        print(f"Ошибка запроса: {e}")
        return []


def get_user_by_id(user_id: int):
    """Получение пользователя по ID."""
    url = f'https://jsonplaceholder.typicode.com/users/{user_id}'
    
    try:
        response = requests.get(url, timeout=10)
        
        if response.status_code == 200:
            return response.json()
        elif response.status_code == 404:
            print(f"Пользователь с ID {user_id} не найден")
            return None
        else:
            print(f"Ошибка: {response.status_code}")
            return None
            
    except requests.exceptions.RequestException as e:
        print(f"Ошибка запроса: {e}")
        return None


# Использование
users = get_users()
user = get_user_by_id(1)
print(user['name'])
3. POST запросы
3.1. Базовый POST запрос
python
import requests

# Данные для отправки
new_post = {
    'title': 'Мой новый пост',
    'body': 'Содержание поста...',
    'userId': 1
}

response = requests.post(
    'https://jsonplaceholder.typicode.com/posts',
    json=new_post  # Автоматически преобразует в JSON и устанавливает Content-Type
)

print(f"Статус: {response.status_code}")  # 201 Created
print(response.json())
3.2. POST с разными форматами данных
python
import requests

# 1. JSON формат (рекомендуемый)
response_json = requests.post(
    'https://httpbin.org/post',
    json={'name': 'Иван', 'age': 25}
)

# 2. Form Data (application/x-www-form-urlencoded)
response_form = requests.post(
    'https://httpbin.org/post',
    data={'name': 'Иван', 'age': 25}
)

# 3. Multipart Form Data (для файлов)
response_file = requests.post(
    'https://httpbin.org/post',
    files={'file': open('document.txt', 'rb')}
)

# 4. Сырые данные
response_raw = requests.post(
    'https://httpbin.org/post',
    data=b'raw data content'
)
3.3. POST с заголовками
python
import requests

headers = {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer secret_token',
    'X-Request-ID': 'req_12345'
}

data = {
    'name': 'Новый продукт',
    'price': 999.99,
    'category': 'electronics'
}

response = requests.post(
    'https://api.example.com/products',
    json=data,
    headers=headers
)

print(response.status_code)
print(response.json())
3.4. Пример: создание пользователя
python
import requests

def create_user(name: str, email: str, username: str) -> dict:
    """Создание нового пользователя через API."""
    url = 'https://jsonplaceholder.typicode.com/users'
    
    user_data = {
        'name': name,
        'email': email,
        'username': username,
        'address': {
            'street': 'ул. Примерная',
            'city': 'Москва'
        }
    }
    
    try:
        response = requests.post(url, json=user_data, timeout=10)
        
        if response.status_code == 201:
            created_user = response.json()
            print(f"✅ Пользователь создан! ID: {created_user['id']}")
            return created_user
        else:
            print(f"❌ Ошибка: {response.status_code}")
            return None
            
    except requests.exceptions.RequestException as e:
        print(f"❌ Ошибка запроса: {e}")
        return None


# Использование
new_user = create_user(
    name="Иван Петров",
    email="ivan@example.com",
    username="ivan_petrov"
)
4. PUT и DELETE запросы
4.1. PUT запрос (полное обновление)
python
import requests

# PUT — заменяет ресурс целиком
updated_data = {
    'id': 1,
    'title': 'Обновлённое название',
    'body': 'Обновлённое содержание',
    'userId': 1
}

response = requests.put(
    'https://jsonplaceholder.typicode.com/posts/1',
    json=updated_data
)

print(f"Статус: {response.status_code}")  # 200 OK
print(response.json())
4.2. PATCH запрос (частичное обновление)
python
import requests

# PATCH — обновляет только указанные поля
patch_data = {
    'title': 'Новое название (только заголовок)'
}

response = requests.patch(
    'https://jsonplaceholder.typicode.com/posts/1',
    json=patch_data
)

print(f"Статус: {response.status_code}")
print(response.json())
4.3. DELETE запрос
python
import requests

# DELETE — удаление ресурса
response = requests.delete('https://jsonplaceholder.typicode.com/posts/1')

print(f"Статус: {response.status_code}")  # 200 OK или 204 No Content

if response.status_code == 204:
    print("✅ Ресурс успешно удалён (пустой ответ)")
elif response.status_code == 200:
    print(f"✅ Ресурс удалён: {response.json()}")
else:
    print(f"❌ Ошибка удаления: {response.status_code}")
4.4. Пример: полный CRUD клиент
python
import requests

class APIClient:
    """Клиент для работы с REST API."""
    
    BASE_URL = 'https://jsonplaceholder.typicode.com'
    
    def __init__(self, base_url: str = None):
        self.base_url = base_url or self.BASE_URL
        self.session = requests.Session()
    
    def get(self, endpoint: str, params: dict = None):
        """GET запрос."""
        url = f"{self.base_url}/{endpoint}"
        response = self.session.get(url, params=params)
        response.raise_for_status()
        return response.json()
    
    def post(self, endpoint: str, data: dict):
        """POST запрос (создание)."""
        url = f"{self.base_url}/{endpoint}"
        response = self.session.post(url, json=data)
        response.raise_for_status()
        return response.json()
    
    def put(self, endpoint: str, data: dict):
        """PUT запрос (полное обновление)."""
        url = f"{self.base_url}/{endpoint}"
        response = self.session.put(url, json=data)
        response.raise_for_status()
        return response.json()
    
    def patch(self, endpoint: str, data: dict):
        """PATCH запрос (частичное обновление)."""
        url = f"{self.base_url}/{endpoint}"
        response = self.session.patch(url, json=data)
        response.raise_for_status()
        return response.json()
    
    def delete(self, endpoint: str):
        """DELETE запрос (удаление)."""
        url = f"{self.base_url}/{endpoint}"
        response = self.session.delete(url)
        response.raise_for_status()
        return response.status_code == 204


# Использование
client = APIClient()

# GET — список постов
posts = client.get('posts', params={'_limit': 5})
print(f"Получено {len(posts)} постов")

# POST — создание поста
new_post = client.post('posts', {
    'title': 'Новый пост',
    'body': 'Содержание...',
    'userId': 1
})
print(f"Создан пост с ID {new_post['id']}")

# PUT — обновление поста
updated = client.put(f"posts/{new_post['id']}", {
    'id': new_post['id'],
    'title': 'Обновлённый пост',
    'body': 'Новое содержание...',
    'userId': 1
})
print(f"Обновлён: {updated['title']}")

# DELETE — удаление поста
client.delete(f"posts/{new_post['id']}")
print("Пост удалён")
5. Обработка ответов
5.1. Проверка статус-кодов
python
import requests

response = requests.get('https://jsonplaceholder.typicode.com/users/1')

# Способ 1: проверка status_code
if response.status_code == 200:
    data = response.json()
    print("Успешно!")
elif response.status_code == 404:
    print("Ресурс не найден")
else:
    print(f"Ошибка: {response.status_code}")

# Способ 2: raise_for_status() — вызывает исключение при ошибке
try:
    response.raise_for_status()
    data = response.json()
except requests.exceptions.HTTPError as e:
    print(f"HTTP ошибка: {e}")
5.2. Парсинг JSON ответа
python
import requests

response = requests.get('https://jsonplaceholder.typicode.com/todos/1')

# response.json() — автоматический парсинг JSON
try:
    data = response.json()
    print(f"Задача: {data['title']}")
    print(f"Выполнена: {data['completed']}")
except requests.exceptions.JSONDecodeError:
    print("Ответ не в JSON формате")
    print(response.text)
5.3. Работа с заголовками ответа
python
import requests

response = requests.get('https://api.github.com')

# Получение заголовков
print(f"Content-Type: {response.headers.get('Content-Type')}")
print(f"Server: {response.headers.get('Server')}")
print(f"Rate limit: {response.headers.get('X-RateLimit-Limit')}")

# Все заголовки
for key, value in response.headers.items():
    print(f"{key}: {value}")
5.4. Обработка ошибок
python
import requests
from requests.exceptions import (
    RequestException, HTTPError, ConnectionError, Timeout, TooManyRedirects
)

def safe_request(url: str, method: str = 'GET', **kwargs):
    """Безопасное выполнение HTTP запроса."""
    try:
        response = requests.request(method, url, timeout=10, **kwargs)
        response.raise_for_status()
        return response
        
    except ConnectionError:
        print("❌ Ошибка подключения: проверьте интернет-соединение")
    except Timeout:
        print("❌ Таймаут: сервер не отвечает")
    except HTTPError as e:
        print(f"❌ HTTP ошибка: {e.response.status_code}")
    except TooManyRedirects:
        print("❌ Слишком много перенаправлений")
    except RequestException as e:
        print(f"❌ Ошибка запроса: {e}")
    
    return None


# Использование
response = safe_request('https://jsonplaceholder.typicode.com/users')
if response:
    print(f"Успешно! Статус: {response.status_code}")
6. Практические примеры
6.1. Работа с публичными API
python
import requests

class WeatherAPI:
    """Клиент для работы с API погоды."""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = 'https://api.openweathermap.org/data/2.5'
    
    def get_current_weather(self, city: str) -> dict:
        """Получение текущей погоды в городе."""
        url = f"{self.base_url}/weather"
        
        params = {
            'q': city,
            'appid': self.api_key,
            'units': 'metric',  # Цельсий
            'lang': 'ru'
        }
        
        try:
            response = requests.get(url, params=params, timeout=10)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            print(f"Ошибка получения погоды: {e}")
            return None
    
    def format_weather(self, weather_data: dict) -> str:
        """Форматирует данные погоды для вывода."""
        if not weather_data:
            return "Данные о погоде недоступны"
        
        city = weather_data['name']
        temp = weather_data['main']['temp']
        feels_like = weather_data['main']['feels_like']
        description = weather_data['weather'][0]['description']
        humidity = weather_data['main']['humidity']
        
        return f"""
        🌍 Город: {city}
        🌡️ Температура: {temp}°C (ощущается как {feels_like}°C)
        📝 Описание: {description}
        💧 Влажность: {humidity}%
        """


# Использование (требуется API ключ)
# weather = WeatherAPI("your_api_key")
# data = weather.get_current_weather("Moscow")
# print(weather.format_weather(data))
6.2. Работа с GitHub API
python
import requests

class GitHubAPI:
    """Клиент для работы с GitHub API."""
    
    def __init__(self, token: str = None):
        self.session = requests.Session()
        self.base_url = 'https://api.github.com'
        
        if token:
            self.session.headers.update({
                'Authorization': f'token {token}',
                'Accept': 'application/vnd.github.v3+json'
            })
    
    def get_user(self, username: str) -> dict:
        """Получение информации о пользователе."""
        url = f"{self.base_url}/users/{username}"
        response = self.session.get(url)
        
        if response.status_code == 200:
            return response.json()
        elif response.status_code == 404:
            print(f"Пользователь {username} не найден")
        else:
            print(f"Ошибка: {response.status_code}")
        
        return None
    
    def get_repos(self, username: str, limit: int = 10) -> list:
        """Получение списка репозиториев пользователя."""
        url = f"{self.base_url}/users/{username}/repos"
        params = {'per_page': limit, 'sort': 'updated'}
        
        response = self.session.get(url, params=params)
        
        if response.status_code == 200:
            return response.json()
        return []
    
    def get_user_info(self, username: str) -> str:
        """Форматирует информацию о пользователе."""
        user = self.get_user(username)
        if not user:
            return "Пользователь не найден"
        
        repos = self.get_repos(username)
        
        return f"""
        📊 Пользователь GitHub: {user['login']}
        👤 Имя: {user.get('name', 'Не указано')}
        📍 Локация: {user.get('location', 'Не указана')}
        📝 Био: {user.get('bio', 'Отсутствует')}
        📁 Репозиториев: {user['public_repos']}
        👥 Подписчиков: {user['followers']}
        📋 Последние репозитории: {', '.join([r['name'] for r in repos[:3]])}
        🔗 Профиль: {user['html_url']}
        """


# Использование
github = GitHubAPI()
info = github.get_user_info('octocat')
print(info)
6.3. Аутентификация
python
import requests

# Basic Auth
response = requests.get(
    'https://api.example.com/private',
    auth=('username', 'password')
)

# Bearer Token
headers = {'Authorization': 'Bearer your_token_here'}
response = requests.get(
    'https://api.example.com/private',
    headers=headers
)

# API Key в query параметрах
params = {'api_key': 'your_api_key'}
response = requests.get(
    'https://api.example.com/data',
    params=params
)

# API Key в заголовках
headers = {'X-API-Key': 'your_api_key'}
response = requests.get(
    'https://api.example.com/data',
    headers=headers
)
6.4. Сессии и куки
python
import requests

# Создание сессии (автоматически сохраняет куки)
session = requests.Session()

# Логин
login_data = {
    'username': 'user',
    'password': 'pass'
}
session.post('https://example.com/login', data=login_data)

# Теперь все запросы будут с куками авторизации
profile = session.get('https://example.com/profile')
posts = session.get('https://example.com/posts')

# Закрытие сессии
session.close()

# Или с контекстным менеджером
with requests.Session() as session:
    session.post('https://example.com/login', data=login_data)
    profile = session.get('https://example.com/profile')
7. Контрольные вопросы
Как установить библиотеку requests?

Как выполнить GET запрос с параметрами?

Чем отличается POST от PUT?

Как передать JSON данные в POST запросе?

Как обработать ошибку 404?

Как получить тело ответа в виде JSON?

Как добавить заголовки к запросу?

Как установить таймаут запроса?

Что делает метод response.raise_for_status()?

Как использовать сессию для сохранения кук?

8. Практическое задание
Задание 1 (базовое)
Напишите функцию get_weather(city), которая получает погоду из публичного API и выводит температуру и описание.

Задание 2 (среднее)
Создайте класс TodoClient для работы с API задач:

get_todos() — получить все задачи

create_todo(title) — создать задачу

complete_todo(todo_id) — отметить как выполненную

delete_todo(todo_id) — удалить задачу

Задание 3 (сложное)
Создайте CLI-утилиту для работы с GitHub API:

python github.py user username — информация о пользователе

python github.py repos username — список репозиториев

python github.py stars username — количество звёзд

9. Шпаргалка
python
# === GET ===
requests.get(url, params={'key': 'value'})

# === POST ===
requests.post(url, json={'key': 'value'})
requests.post(url, data={'key': 'value'})

# === PUT ===
requests.put(url, json={'key': 'value'})

# === DELETE ===
requests.delete(url)

# === ЗАГОЛОВКИ ===
headers = {'Authorization': 'Bearer token'}
requests.get(url, headers=headers)

# === ТАЙМАУТ ===
requests.get(url, timeout=5)

# === СЕССИЯ ===
with requests.Session() as session:
    session.get(url)

# === ОБРАБОТКА ОТВЕТА ===
response.status_code
response.headers
response.text
response.json()
response.raise_for_status()

# === ОШИБКИ ===
from requests.exceptions import RequestException, HTTPError, ConnectionError, Timeout
Итог лекции
Вы сегодня:

✅ Изучили библиотеку requests для HTTP-запросов

✅ Освоили GET, POST, PUT, DELETE методы

✅ Научились работать с параметрами, заголовками и телом запроса

✅ Изучили обработку ответов и ошибок

✅ Создали клиентов для работы с реальными API

Теперь вы можете интегрировать любые веб-API в свои Python-приложения!
