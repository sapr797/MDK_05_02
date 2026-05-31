# Тема 2.16. Введение в REST API. HTTP-протокол, методы, статус-коды

**Цель лекции:**  
Изучить основы REST API, понять принципы работы HTTP-протокола, освоить основные HTTP-методы и статус-коды, научиться проектировать RESTful API.

> Главная мысль: **API — это мост между приложениями. REST — это архитектурный стиль, который делает этот мост понятным и предсказуемым.**

---

## Содержание

1. [Что такое API и REST API](#1-что-такое-api-и-rest-api)
2. [HTTP-протокол](#2-http-протокол)
3. [HTTP-методы](#3-http-методы)
4. [Статус-коды HTTP](#4-статус-коды-http)
5. [RESTful принципы](#5-restful-принципы)
6. [Форматы данных: JSON](#6-форматы-данных-json)
7. [Практические примеры](#7-практические-примеры)
8. [Контрольные вопросы](#8-контрольные-вопросы)
9. [Практическое задание](#9-практическое-задание)
10. [Шпаргалка](#10-шпаргалка)

---

## 1. Что такое API и REST API

### 1.1. Что такое API

**API (Application Programming Interface)** — это набор правил и инструментов, с помощью которых одна программа может взаимодействовать с другой.
┌─────────────┐ ┌─────────────┐
│ Клиент │ API запрос │ Сервер │
│ (приложение)│ ──────────────────► │ (сервис) │
│ │ │ │
│ │ API ответ │ │
│ │ ◄────────────────── │ │
└─────────────┘ └─────────────┘

text

**Примеры API:**
- API погоды для получения прогноза
- API Telegram для отправки сообщений
- API Google Maps для отображения карт

### 1.2. Что такое REST API

**REST (Representational State Transfer)** — это архитектурный стиль для построения API.

**Основные характеристики REST API:**

| Характеристика | Описание |
|----------------|----------|
| **Клиент-сервер** | Разделение интерфейса и логики |
| **Отсутствие состояния** | Каждый запрос содержит всю необходимую информацию |
| **Кэширование** | Ответы могут кэшироваться |
| **Единообразие интерфейса** | Стандартные методы и форматы |
| **Слои** | Промежуточные серверы не влияют на взаимодействие |
| **Код по требованию** | Сервер может передавать исполняемый код |

### 1.3. REST vs SOAP

| Характеристика | REST | SOAP |
|----------------|------|------|
| Протокол | HTTP | HTTP, SMTP, TCP |
| Формат | JSON, XML, HTML | XML |
| Простота | Высокая | Низкая |
| Кэширование | Да | Нет |
| Производительность | Высокая | Низкая |

---

## 2. HTTP-протокол

### 2.1. Структура HTTP запроса
┌─────────────────────────────────────────────────────────────────┐
│ HTTP ЗАПРОС │
├─────────────────────────────────────────────────────────────────┤
│ СТРОКА ЗАПРОСА │
│ GET /users/123 HTTP/1.1 │
├─────────────────────────────────────────────────────────────────┤
│ ЗАГОЛОВКИ │
│ Host: api.example.com │
│ User-Agent: Mozilla/5.0 │
│ Accept: application/json │
│ Authorization: Bearer token123 │
│ Content-Type: application/json │
├─────────────────────────────────────────────────────────────────┤
│ ТЕЛО (опционально) │
│ {"name": "Иван", "email": "ivan@example.com"} │
└─────────────────────────────────────────────────────────────────┘

text

### 2.2. Структура HTTP ответа
┌─────────────────────────────────────────────────────────────────┐
│ HTTP ОТВЕТ │
├─────────────────────────────────────────────────────────────────┤
│ СТРОКА СТАТУСА │
│ HTTP/1.1 200 OK │
├─────────────────────────────────────────────────────────────────┤
│ ЗАГОЛОВКИ │
│ Content-Type: application/json │
│ Content-Length: 123 │
│ Cache-Control: max-age=3600 │
├─────────────────────────────────────────────────────────────────┤
│ ТЕЛО │
│ {"id": 123, "name": "Иван", "email": "ivan@example.com"} │
└─────────────────────────────────────────────────────────────────┘

text

### 2.3. Основные заголовки HTTP

| Заголовок | Назначение | Пример |
|-----------|------------|--------|
| `Host` | Имя сервера | `api.example.com` |
| `User-Agent` | Информация о клиенте | `Mozilla/5.0` |
| `Accept` | Ожидаемый формат ответа | `application/json` |
| `Content-Type` | Тип данных в теле запроса | `application/json` |
| `Authorization` | Токен авторизации | `Bearer token123` |
| `Content-Length` | Длина тела сообщения | `123` |

---

## 3. HTTP-методы

### 3.1. Основные методы

| Метод | CRUD операция | Описание | Безопасность | Идемпотентность |
|-------|---------------|----------|--------------|-----------------|
| `GET` | READ | Получение данных | ✅ | ✅ |
| `POST` | CREATE | Создание ресурса | ❌ | ❌ |
| `PUT` | UPDATE (полное) | Обновление ресурса | ❌ | ✅ |
| `PATCH` | UPDATE (частичное) | Частичное обновление | ❌ | ❌ |
| `DELETE` | DELETE | Удаление ресурса | ❌ | ✅ |
| `HEAD` | - | Получение только заголовков | ✅ | ✅ |
| `OPTIONS` | - | Получение доступных методов | ✅ | ✅ |

### 3.2. Примеры использования

```http
# GET — получение списка пользователей
GET /users HTTP/1.1
Host: api.example.com

# GET — получение конкретного пользователя
GET /users/123 HTTP/1.1
Host: api.example.com

# POST — создание нового пользователя
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
    "name": "Иван",
    "email": "ivan@example.com"
}

# PUT — полное обновление пользователя
PUT /users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
    "id": 123,
    "name": "Иван Петров",
    "email": "ivan@example.com"
}

# PATCH — частичное обновление
PATCH /users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
    "name": "Иван Петров"
}

# DELETE — удаление пользователя
DELETE /users/123 HTTP/1.1
Host: api.example.com
3.3. Идемпотентность
Идемпотентность — свойство операции, при котором многократное выполнение даёт тот же результат, что и однократное.

Метод	Идемпотентность	Объяснение
GET	✅	Не изменяет состояние
PUT	✅	При повторном вызове ресурс остаётся таким же
DELETE	✅	Повторное удаление несуществующего ресурса не меняет состояние
POST	❌	Каждый вызов создаёт новый ресурс
PATCH	❌	Может зависеть от текущего состояния
4. Статус-коды HTTP
4.1. Классификация
Диапазон	Категория	Описание
1xx	Информационные	Запрос получен, обработка продолжается
2xx	Успех	Запрос успешно обработан
3xx	Перенаправление	Требуется дополнительное действие
4xx	Ошибка клиента	Ошибка в запросе клиента
5xx	Ошибка сервера	Ошибка на стороне сервера
4.2. Наиболее часто используемые коды
2xx — Успех
Код	Название	Описание
200	OK	Запрос успешно выполнен
201	Created	Ресурс успешно создан
202	Accepted	Запрос принят в обработку
204	No Content	Успешно, но тело ответа пустое
3xx — Перенаправление
Код	Название	Описание
301	Moved Permanently	Ресурс перемещён навсегда
302	Found	Временное перенаправление
304	Not Modified	Ресурс не изменился (кэш)
4xx — Ошибка клиента
Код	Название	Описание
400	Bad Request	Неверный формат запроса
401	Unauthorized	Требуется аутентификация
403	Forbidden	Доступ запрещён
404	Not Found	Ресурс не найден
405	Method Not Allowed	Метод не поддерживается
409	Conflict	Конфликт данных
422	Unprocessable Entity	Ошибка валидации
429	Too Many Requests	Превышен лимит запросов
5xx — Ошибка сервера
Код	Название	Описание
500	Internal Server Error	Внутренняя ошибка сервера
501	Not Implemented	Метод не реализован
502	Bad Gateway	Ошибка шлюза
503	Service Unavailable	Сервис временно недоступен
504	Gateway Timeout	Таймаут шлюза
4.3. Примеры использования статус-кодов
python
# Успешное получение данных
GET /users/123
Response: 200 OK

# Успешное создание
POST /users
Response: 201 Created
Location: /users/456

# Ресурс не найден
GET /users/999
Response: 404 Not Found

# Ошибка валидации
POST /users
{
    "email": "invalid"
}
Response: 422 Unprocessable Entity
{
    "errors": {
        "email": "Invalid email format"
    }
}

# Нет прав доступа
DELETE /users/123
Response: 403 Forbidden
5. RESTful принципы
5.1. Ресурсы и их идентификация
Ресурс — это любой объект, к которому можно обратиться через API.

text
# Правильные URL (ресурсы — существительные)
GET  /users          # список пользователей
GET  /users/123      # конкретный пользователь
GET  /users/123/posts # посты пользователя
GET  /posts/456      # конкретный пост

# Неправильные URL (глаголы в URL)
GET  /getUsers
POST /createUser
GET  /getUserById?id=123
5.2. Использование HTTP-методов
Действие	Метод	URL
Список ресурсов	GET	/users
Получение ресурса	GET	/users/{id}
Создание ресурса	POST	/users
Полное обновление	PUT	/users/{id}
Частичное обновление	PATCH	/users/{id}
Удаление	DELETE	/users/{id}
5.3. Версионирование API
http
# В URL
GET /v1/users
GET /v2/users

# В заголовке
GET /users
Accept: application/vnd.myapi.v1+json

# В параметре запроса
GET /users?version=1
5.4. Пагинация, фильтрация, сортировка
http
# Пагинация
GET /users?page=2&limit=20

# Фильтрация
GET /users?status=active&age_gt=18

# Сортировка
GET /users?sort=-created_at,asc

# Поиск
GET /users?search=ivan

# Комбинация
GET /users?page=1&limit=10&status=active&sort=-created_at
Ответ с пагинацией:

json
{
    "data": [...],
    "meta": {
        "page": 1,
        "limit": 10,
        "total": 42,
        "pages": 5
    },
    "links": {
        "first": "/users?page=1",
        "prev": null,
        "next": "/users?page=2",
        "last": "/users?page=5"
    }
}
6. Форматы данных: JSON
6.1. Структура JSON
json
{
    "id": 123,
    "name": "Иван Петров",
    "email": "ivan@example.com",
    "age": 25,
    "is_active": true,
    "address": {
        "city": "Москва",
        "street": "Тверская",
        "house": 10
    },
    "hobbies": ["чтение", "спорт", "музыка"],
    "created_at": "2024-01-15T14:30:00Z"
}
6.2. Примеры API ответов
Успешный ответ (200 OK):

json
{
    "success": true,
    "data": {
        "id": 123,
        "name": "Иван Петров",
        "email": "ivan@example.com"
    }
}
Список ресурсов (200 OK):

json
{
    "success": true,
    "data": [
        {"id": 1, "name": "Иван"},
        {"id": 2, "name": "Мария"},
        {"id": 3, "name": "Петр"}
    ],
    "meta": {
        "total": 3
    }
}
Ошибка валидации (422 Unprocessable Entity):

json
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Ошибка валидации данных",
        "details": {
            "email": "Поле email обязательно",
            "age": "Значение должно быть не менее 18"
        }
    }
}
Ошибка аутентификации (401 Unauthorized):

json
{
    "success": false,
    "error": {
        "code": "UNAUTHORIZED",
        "message": "Требуется авторизация"
    }
}
7. Практические примеры
7.1. Пример API запроса с помощью curl
bash
# GET запрос
curl -X GET https://api.example.com/users/123

# POST запрос с телом
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token123" \
  -d '{"name":"Иван","email":"ivan@example.com"}'

# PUT запрос
curl -X PUT https://api.example.com/users/123 \
  -H "Content-Type: application/json" \
  -d '{"name":"Иван Петров","email":"ivan@example.com"}'

# DELETE запрос
curl -X DELETE https://api.example.com/users/123
7.2. Пример API запроса на Python
python
import requests

# GET запрос
response = requests.get('https://api.example.com/users/123')
if response.status_code == 200:
    user = response.json()
    print(f"Пользователь: {user['name']}")
else:
    print(f"Ошибка: {response.status_code}")

# POST запрос
data = {
    "name": "Иван",
    "email": "ivan@example.com"
}
response = requests.post(
    'https://api.example.com/users',
    json=data,
    headers={'Authorization': 'Bearer token123'}
)

if response.status_code == 201:
    created_user = response.json()
    print(f"Создан пользователь: {created_user['id']}")
else:
    print(f"Ошибка: {response.status_code} - {response.text}")
7.3. Обработка ошибок
python
import requests

def get_user(user_id: int):
    try:
        response = requests.get(f'https://api.example.com/users/{user_id}', timeout=5)
        
        if response.status_code == 200:
            return response.json()
        elif response.status_code == 404:
            print(f"Пользователь {user_id} не найден")
        elif response.status_code == 401:
            print("Требуется авторизация")
        elif response.status_code == 429:
            print("Превышен лимит запросов. Попробуйте позже.")
        else:
            print(f"Ошибка: {response.status_code}")
        
        return None
        
    except requests.exceptions.Timeout:
        print("Таймаут запроса")
    except requests.exceptions.ConnectionError:
        print("Ошибка соединения")
    except requests.exceptions.RequestException as e:
        print(f"Ошибка запроса: {e}")
    
    return None
8. Контрольные вопросы
Что такое API и для чего он нужен?

Что означает аббревиатура REST? Какие принципы лежат в основе REST?

Какие основные HTTP-методы существуют? Какие операции они выполняют?

Что такое идемпотентность? Какие HTTP-методы идемпотентны?

Какие группы статус-кодов HTTP существуют? Приведите примеры.

Что означает статус-код 404? 201? 401? 500?

Как правильно проектировать URL для REST API?

Как реализовать пагинацию в REST API?

Какой формат данных чаще всего используется в REST API?

Чем отличается PUT от PATCH?

9. Практическое задание
Задание 1 (базовое)
Напишите функцию make_api_request(url, method, data=None), которая выполняет HTTP-запрос и обрабатывает возможные ошибки.

Задание 2 (среднее)
Создайте класс APIClient для работы с REST API, который поддерживает методы get, post, put, delete и обрабатывает статус-коды.

Задание 3 (сложное)
Спроектируйте REST API для интернет-магазина:

Товары (GET, POST, PUT, DELETE)

Заказы (GET, POST, PUT, DELETE)

Пользователи (GET, POST, PUT, DELETE)

Корзина (GET, POST, DELETE)

Опишите все эндпоинты с методами, параметрами и ожидаемыми статус-кодами.

10. Шпаргалка
http
# === ОСНОВНЫЕ МЕТОДЫ ===
GET     /resource          # получить список
GET     /resource/{id}     # получить один
POST    /resource          # создать
PUT     /resource/{id}     # полностью обновить
PATCH   /resource/{id}     # частично обновить
DELETE  /resource/{id}     # удалить

# === ОСНОВНЫЕ СТАТУС-КОДЫ ===
200 OK                     # успех
201 Created               # создано
204 No Content            # успех (без тела)
400 Bad Request           # неверный запрос
401 Unauthorized          # не авторизован
403 Forbidden             # доступ запрещён
404 Not Found             # не найдено
422 Unprocessable Entity  # ошибка валидации
429 Too Many Requests     # слишком много запросов
500 Internal Server Error # ошибка сервера

# === ЗАГОЛОВКИ ===
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>
Итог лекции
Вы сегодня:

✅ Изучили основы REST API

✅ Познакомились с HTTP-протоколом и его структурой

✅ Освоили основные HTTP-методы и их назначение

✅ Изучили статус-коды и их группы

✅ Поняли RESTful принципы проектирования API

Теперь вы знаете, как устроены API и как с ними работать!
