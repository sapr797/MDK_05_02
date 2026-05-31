# Контрольная работа №4 (по темам 2.19-2.21)

**Дисциплина:** Основы программирования на Python  
**Курс:** 1 курс, СПО  
**Время выполнения:** 90 минут  
**Максимальный балл:** 100

---

## Содержание

1. [Структура контрольной работы](#1-структура-контрольной-работы)
2. [Перечень тем для контроля](#2-перечень-тем-для-контроля)
3. [Вариант 0 (демонстрационный)](#3-вариант-0-демонстрационный)
4. [25 вариантов заданий](#4-25-вариантов-заданий)
5. [Критерии оценки](#5-критерии-оценки)
6. [Лист ответов](#6-лист-ответов)

---

## 1. Структура контрольной работы

Каждый вариант контрольной работы состоит из **трёх частей**:

| Часть | Разделы | Тип заданий | Баллы |
|-------|---------|-------------|-------|
| **Часть А** (тестовая) | 2.19-2.21 | 10 вопросов с выбором ответа | 30 |
| **Часть В** (практическая) | 2.19-2.20 | Написание кода (30-40 строк) | 40 |
| **Часть С** (аналитическая) | 2.21 | Проектирование системы | 30 |

---

## 2. Перечень тем для контроля

| Раздел | Тема | Ключевые понятия |
|--------|------|------------------|
| **2.19** | Разработка HTTP-клиента | requests, GET, POST, PUT, DELETE |
| **2.20** | Рендеринг HTML на сервере | Jinja2, шаблоны, наследование |
| **2.21** | Клиентские запросы | Fetch API, async/await, JSON |

---

## 3. Вариант 0 (демонстрационный)

**Назначение:** преподаватель выполняет этот вариант на занятии, показывая структуру ответов.

### Часть А. Тестовые задания (30 баллов)

**Выберите один правильный ответ.**

**А1.** Какой метод библиотеки requests выполняет DELETE запрос?

```text
А) requests.get()
Б) requests.post()
В) requests.put()
Г) requests.delete()
Правильный ответ: Г

А2. Какой метод JavaScript Fetch API используется для отправки данных в формате JSON?

text
А) fetch(url)
Б) fetch(url, {method: 'GET'})
В) fetch(url, {method: 'POST', body: JSON.stringify(data)})
Г) fetch(url, {method: 'PUT'})
Правильный ответ: В

А3. Какой синтаксис используется в Jinja2 для вывода переменной?

text
А) [[ variable ]]
Б) {{ variable }}
В) {% variable %}
Г) {# variable #}
Правильный ответ: Б

А4. Как получить JSON-ответ в библиотеке requests?

text
А) response.text()
Б) response.content()
В) response.json()
Г) response.parse()
Правильный ответ: В

А5. Какой заголовок используется для авторизации с Bearer токеном?

text
А) X-API-Key
Б) Authentication
В) Authorization
Г) Token
Правильный ответ: В

А6. Какой метод Fetch API используется для выполнения PUT запроса?

text
А) fetch(url, {method: 'PUT'})
Б) fetch(url, {method: 'PUT', body: data})
В) fetch(url, {method: 'PUT'}) и body
Г) Все варианты верны
Правильный ответ: Г

А7. Какой блок в Jinja2 используется для цикла?

text
А) {% if %}
Б) {% for %}
В) {% while %}
Г) {% loop %}
Правильный ответ: Б

А8. Как обработать ошибку 404 в requests?

text
А) response.raise_for_status()
Б) if response.status_code == 404
В) try/except
Г) Все варианты верны
Правильный ответ: Г

А9. Какой атрибут ответа Fetch API содержит статус HTTP?

text
А) response.status
Б) response.statusCode
В) response.code
Г) response.ok
Правильный ответ: А

А10. Какой метод Jinja2 используется для наследования шаблонов?

text
А) {% include %}
Б) {% import %}
В) {% extends %}
Г) {% block %}
Правильный ответ: В

Часть В. Практическое задание (40 баллов)
Задание: Напишите функцию на Python, которая:

Выполняет GET запрос к API https://jsonplaceholder.typicode.com/posts

Получает список постов

Фильтрует посты по userId (параметр функции)

Возвращает отфильтрованные посты в виде списка словарей

Эталонное решение:

python
import requests
from typing import List, Dict, Any


def get_posts_by_user(user_id: int) -> List[Dict[str, Any]]:
    """
    Получает посты пользователя из API.
    
    Args:
        user_id: ID пользователя
    
    Returns:
        Список постов пользователя
    """
    url = "https://jsonplaceholder.typicode.com/posts"
    
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        
        all_posts = response.json()
        filtered_posts = [post for post in all_posts if post.get('userId') == user_id]
        
        return filtered_posts
        
    except requests.exceptions.RequestException as e:
        print(f"Ошибка при запросе: {e}")
        return []


# Пример использования
if __name__ == "__main__":
    posts = get_posts_by_user(1)
    print(f"Найдено постов: {len(posts)}")
    for post in posts[:3]:
        print(f"  {post['title']}")
Часть С. Аналитическое задание (30 баллов)
Задание: Спроектируйте систему для отображения списка задач с использованием FastAPI + Jinja2 + Fetch API.

Требования:

FastAPI эндпоинт возвращает HTML страницу с шаблоном

JavaScript через Fetch API загружает список задач

Jinja2 шаблон отображает задачи

Возможность добавления новой задачи через форму

Эталонный ответ:

Схема взаимодействия:

text
┌─────────┐     GET /          ┌─────────┐
│ Браузер │ ─────────────────► │ FastAPI │
│         │ ◄───────────────── │         │
└─────────┘     HTML (Jinja2)  └─────────┘
     │                                 │
     │ GET /api/tasks                   │
     │─────────────────────────────────►│
     │◄─────────────────────────────────│
     │         JSON                     │
     │                                 │
     │ POST /api/tasks                  │
     │─────────────────────────────────►│
     │◄─────────────────────────────────│
     │         201 Created              │
Код FastAPI:

python
from fastapi import FastAPI, Request, Form
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles
from pydantic import BaseModel
from typing import List

app = FastAPI()
templates = Jinja2Templates(directory="templates")
app.mount("/static", StaticFiles(directory="static"), name="static")

tasks_db = []
counter = 1


class Task(BaseModel):
    id: int
    title: str
    completed: bool = False


@app.get("/")
async def home(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})


@app.get("/api/tasks", response_model=List[Task])
async def get_tasks():
    return tasks_db


@app.post("/api/tasks", response_model=Task, status_code=201)
async def create_task(title: str = Form(...)):
    global counter
    task = Task(id=counter, title=title)
    tasks_db.append(task)
    counter += 1
    return task
Шаблон index.html:

html
<!DOCTYPE html>
<html>
<head>
    <title>Task Manager</title>
</head>
<body>
    <h1>Мои задачи</h1>
    
    <form id="taskForm">
        <input type="text" id="title" placeholder="Новая задача" required>
        <button type="submit">Добавить</button>
    </form>
    
    <ul id="taskList"></ul>
    
    <script>
        async function loadTasks() {
            const response = await fetch('/api/tasks');
            const tasks = await response.json();
            
            const list = document.getElementById('taskList');
            list.innerHTML = tasks.map(task => 
                `<li>${task.title} ${task.completed ? '✅' : '⏳'}</li>`
            ).join('');
        }
        
        document.getElementById('taskForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            const title = document.getElementById('title').value;
            
            await fetch('/api/tasks', {
                method: 'POST',
                headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                body: `title=${encodeURIComponent(title)}`
            });
            
            document.getElementById('title').value = '';
            loadTasks();
        });
        
        loadTasks();
    </script>
</body>
</html>
4. 25 вариантов заданий
Варианты 1-8 (Базовый уровень)
Вариант	Часть А (темы)	Часть В (задание)	Часть С (анализ)
1	2.19-2.20	GET запрос, парсинг JSON	FastAPI + Jinja2
2	2.20-2.21	POST запрос, создание	Fetch API + шаблон
3	2.19-2.21	PUT запрос, обновление	Три компонента
4	2.19-2.20	DELETE запрос	API + шаблон
5	2.20-2.21	Фильтрация данных	Fetch + Jinja2
6	2.19-2.21	Обработка ошибок	Полный цикл
7	2.19-2.20	Авторизация Bearer	API документация
8	2.20-2.21	Пагинация	Интерактивность
Варианты 9-17 (Средний уровень)
Вариант	Часть А	Часть В	Часть С
9	2.19-2.20	Параллельные запросы	Сложная фильтрация
10	2.20-2.21	Вложенные шаблоны	Сортировка
11	2.19-2.21	Сессии и куки	Поиск
12	2.19-2.20	Retry механизм	Пагинация
13	2.20-2.21	Макросы Jinja2	Бесконечный скролл
14	2.19-2.21	Таймауты	Ленивая загрузка
15	2.19-2.20	Аутентификация	Формы
16	2.20-2.21	Наследование шаблонов	WebSocket
17	2.19-2.21	Потоковые ответы	Дашборд
Варианты 18-25 (Сложный уровень)
Вариант	Тема	Особенности
18	Агрегатор API	Несколько источников
19	Дашборд	Графики, метрики
20	Чат	Real-time обновления
21	ETL интерфейс	Прогресс-бар
22	Админ-панель	CRUD + фильтры
23	Экспорт данных	CSV/Excel
24	Импорт данных	Drag & drop
25	Мониторинг	Логи, алерты
5. Критерии оценки
Оценка	Баллы	Требования
5 (отлично)	90-100	Часть А: 9-10 правильных ответов. Часть В: код работает, есть обработка ошибок. Часть С: полное решение
4 (хорошо)	70-89	Часть А: 7-8 правильных ответов. Часть В: код работает с недочётами
3 (удовлетворительно)	50-69	Часть А: 5-6 правильных ответов. Часть В: код с ошибками
2 (неудовлетворительно)	<50	Часть А: менее 5 правильных ответов. Часть В: код не работает
6. Лист ответов (шаблон)
text
ФИО студента: _________________________________
Группа: _________________________________
Вариант №: _________________________________
Дата: _________________________________

=============================================
ЧАСТЬ А. ТЕСТОВЫЕ ЗАДАНИЯ (30 баллов)
=============================================

| Вопрос | А1 | А2 | А3 | А4 | А5 | А6 | А7 | А8 | А9 | А10 |
|--------|----|----|----|----|----|----|----|----|----|-----|
| Ответ  |    |    |    |    |    |    |    |    |    |     |

=============================================
ЧАСТЬ В. ПРАКТИЧЕСКОЕ ЗАДАНИЕ (40 баллов)
=============================================

(Код прилагается)

=============================================
ЧАСТЬ С. АНАЛИТИЧЕСКОЕ ЗАДАНИЕ (30 баллов)
=============================================

Компоненты системы:
1. _________________________________
2. _________________________________
3. _________________________________

Схема взаимодействия:
_________________________________

=============================================
ИТОГО: _____ / 100 баллов
Оценка: _____
Подпись преподавателя: _____________
=============================================
