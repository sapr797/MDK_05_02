# Тема 2.21. Клиентские запросы (Fetch API)

**Цель лекции:**  
Изучить Fetch API для выполнения асинхронных HTTP-запросов из браузера, освоить методы GET, POST, PUT, DELETE, работу с JSON, обработку ошибок и заголовков.

> Главная мысль: **Fetch API — современная замена XMLHttpRequest для взаимодействия с сервером без перезагрузки страницы.**

---

## Содержание

1. [Введение в Fetch API](#1-введение-в-fetch-api)
2. [GET запросы](#2-get-запросы)
3. [POST запросы](#3-post-запросы)
4. [PUT и DELETE запросы](#4-put-и-delete-запросы)
5. [Обработка ошибок](#5-обработка-ошибок)
6. [Работа с заголовками](#6-работа-с-заголовками)
7. [Асинхронность: async/await](#7-асинхронность-asyncawait)
8. [Практические примеры](#8-практические-примеры)
9. [Контрольные вопросы](#9-контрольные-вопросы)
10. [Практическое задание](#10-практическое-задание)
11. [Шпаргалка](#11-шпаргалка)

---

## 1. Введение в Fetch API

### 1.1. Что такое Fetch API

**Fetch API** — это современный интерфейс JavaScript для выполнения HTTP-запросов к серверу.

**Преимущества Fetch:**

| Характеристика | XMLHttpRequest | Fetch API |
|----------------|----------------|-----------|
| Синтаксис | Громоздкий | Простой, на промисах |
| Асинхронность | Callback | Promise / async/await |
| Работа с JSON | manual | `response.json()` |
| Кэширование | ❌ | ✅ |
| Режим | Только запросы | Запросы и ответы |

### 1.2. Базовый синтаксис

```javascript
// Простой GET запрос
fetch('https://api.example.com/data')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error('Ошибка:', error));

// С async/await (рекомендуется)
async function getData() {
    try {
        const response = await fetch('https://api.example.com/data');
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error('Ошибка:', error);
    }
}
1.3. Структура ответа (Response)
javascript
const response = await fetch(url);

// Свойства ответа
response.ok           // true если статус 200-299
response.status       // HTTP статус (200, 404, 500)
response.statusText   // Текст статуса ("OK", "Not Found")
response.headers      // Заголовки ответа
response.url          // URL ответа

// Методы для получения тела ответа
await response.json()     // Парсинг JSON
await response.text()     // Получение текста
await response.blob()     // Получение Blob (изображения)
await response.formData() // Получение FormData
await response.arrayBuffer() // Получение ArrayBuffer
2. GET запросы
2.1. Простой GET запрос
javascript
// Получение списка пользователей
fetch('https://jsonplaceholder.typicode.com/users')
    .then(response => response.json())
    .then(users => {
        console.log('Пользователи:', users);
        users.forEach(user => {
            console.log(`${user.name} - ${user.email}`);
        });
    })
    .catch(error => console.error('Ошибка:', error));
2.2. GET с параметрами (query string)
javascript
// GET с параметрами (через URLSearchParams)
const params = new URLSearchParams({
    userId: 1,
    _limit: 5
});

fetch(`https://jsonplaceholder.typicode.com/posts?${params}`)
    .then(response => response.json())
    .then(posts => {
        console.log(`Получено ${posts.length} постов`);
        posts.forEach(post => {
            console.log(post.title);
        });
    });

// Пример: получение поста по ID
async function getPost(id) {
    const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`);
    
    if (!response.ok) {
        throw new Error(`Пост ${id} не найден`);
    }
    
    const post = await response.json();
    return post;
}

// Использование
getPost(1).then(post => {
    console.log('Заголовок:', post.title);
});
2.3. GET с заголовками
javascript
// GET с кастомными заголовками
fetch('https://api.github.com/user', {
    method: 'GET',
    headers: {
        'Authorization': 'token YOUR_GITHUB_TOKEN',
        'Accept': 'application/vnd.github.v3+json',
        'User-Agent': 'MyApp/1.0'
    }
})
.then(response => response.json())
.then(user => console.log(user.login))
.catch(error => console.error(error));
3. POST запросы
3.1. Базовый POST запрос
javascript
// Отправка JSON данных
const newPost = {
    title: 'Новый пост',
    body: 'Содержание поста...',
    userId: 1
};

fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(newPost)
})
.then(response => response.json())
.then(data => {
    console.log('Создан пост с ID:', data.id);
    console.log(data);
})
.catch(error => console.error('Ошибка:', error));
3.2. POST с async/await
javascript
async function createPost(title, body, userId) {
    try {
        const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                title: title,
                body: body,
                userId: userId
            })
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ошибка: ${response.status}`);
        }
        
        const newPost = await response.json();
        console.log('Пост создан:', newPost);
        return newPost;
        
    } catch (error) {
        console.error('Ошибка создания поста:', error);
        throw error;
    }
}

// Использование
createPost('Мой заголовок', 'Текст поста...', 1);
3.3. Отправка FormData
javascript
// Отправка данных формы
async function submitForm(event) {
    event.preventDefault();
    
    const formData = new FormData(event.target);
    
    try {
        const response = await fetch('/api/submit', {
            method: 'POST',
            body: formData
            // Content-Type не указываем — Fetch сам установит multipart/form-data
        });
        
        if (response.ok) {
            const result = await response.json();
            console.log('Успешно отправлено:', result);
        } else {
            console.error('Ошибка отправки:', response.status);
        }
    } catch (error) {
        console.error('Ошибка:', error);
    }
}

// HTML форма
<form id="myForm" onsubmit="submitForm(event)">
    <input type="text" name="name" placeholder="Имя">
    <input type="email" name="email" placeholder="Email">
    <button type="submit">Отправить</button>
</form>
3.4. Отправка файлов
javascript
// Загрузка файла на сервер
async function uploadFile(fileInput) {
    const file = fileInput.files[0];
    if (!file) return;
    
    const formData = new FormData();
    formData.append('file', file);
    formData.append('name', file.name);
    
    try {
        const response = await fetch('/upload', {
            method: 'POST',
            body: formData
        });
        
        if (response.ok) {
            const result = await response.json();
            console.log('Файл загружен:', result.url);
            return result;
        } else {
            console.error('Ошибка загрузки');
        }
    } catch (error) {
        console.error('Ошибка:', error);
    }
}

// HTML
<input type="file" id="fileInput" onchange="uploadFile(this)">
4. PUT и DELETE запросы
4.1. PUT запрос (полное обновление)
javascript
// Обновление поста
async function updatePost(id, updatedData) {
    try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                id: id,
                title: updatedData.title,
                body: updatedData.body,
                userId: updatedData.userId
            })
        });
        
        if (!response.ok) {
            throw new Error(`Ошибка обновления: ${response.status}`);
        }
        
        const updatedPost = await response.json();
        console.log('Пост обновлён:', updatedPost);
        return updatedPost;
        
    } catch (error) {
        console.error('Ошибка:', error);
    }
}

// Использование
updatePost(1, {
    title: 'Обновлённый заголовок',
    body: 'Новое содержание',
    userId: 1
});
4.2. PATCH запрос (частичное обновление)
javascript
// Частичное обновление (только заголовок)
async function patchPostTitle(id, newTitle) {
    try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
            method: 'PATCH',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                title: newTitle
            })
        });
        
        const updatedPost = await response.json();
        console.log('Заголовок обновлён:', updatedPost.title);
        return updatedPost;
        
    } catch (error) {
        console.error('Ошибка:', error);
    }
}
4.3. DELETE запрос
javascript
// Удаление поста
async function deletePost(id) {
    try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
            method: 'DELETE'
        });
        
        if (response.ok) {
            console.log(`Пост ${id} удалён`);
            return true;
        } else {
            console.error(`Ошибка удаления: ${response.status}`);
            return false;
        }
    } catch (error) {
        console.error('Ошибка:', error);
        return false;
    }
}

// Использование
deletePost(1).then(success => {
    if (success) {
        // Обновляем список постов
        loadPosts();
    }
});
5. Обработка ошибок
5.1. Проверка статуса ответа
javascript
async function fetchWithErrorHandling(url, options = {}) {
    try {
        const response = await fetch(url, options);
        
        // Проверка успешности запроса
        if (!response.ok) {
            // Обработка разных статусов
            switch (response.status) {
                case 400:
                    throw new Error('Неверный запрос (400)');
                case 401:
                    throw new Error('Требуется авторизация (401)');
                case 403:
                    throw new Error('Доступ запрещён (403)');
                case 404:
                    throw new Error('Ресурс не найден (404)');
                case 500:
                    throw new Error('Внутренняя ошибка сервера (500)');
                default:
                    throw new Error(`HTTP ошибка: ${response.status}`);
            }
        }
        
        const data = await response.json();
        return { success: true, data };
        
    } catch (error) {
        console.error('Ошибка запроса:', error);
        return { success: false, error: error.message };
    }
}

// Использование
const result = await fetchWithErrorHandling('https://api.example.com/users');
if (result.success) {
    console.log('Данные:', result.data);
} else {
    console.error('Ошибка:', result.error);
}
5.2. Обработка сетевых ошибок
javascript
async function robustFetch(url, options = {}, retries = 3) {
    for (let i = 0; i < retries; i++) {
        try {
            const response = await fetch(url, options);
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }
            
            return await response.json();
            
        } catch (error) {
            console.log(`Попытка ${i + 1} из ${retries} не удалась:`, error.message);
            
            if (i === retries - 1) {
                throw new Error(`Не удалось выполнить запрос после ${retries} попыток`);
            }
            
            // Ждём перед повторной попыткой (экспоненциальная задержка)
            await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
        }
    }
}

// Использование
try {
    const data = await robustFetch('https://api.example.com/data', {}, 5);
    console.log('Данные получены:', data);
} catch (error) {
    console.error('Финальная ошибка:', error.message);
}
6. Работа с заголовками
6.1. Установка заголовков
javascript
// Заголовки запроса
const headers = new Headers();
headers.append('Content-Type', 'application/json');
headers.append('Authorization', 'Bearer token123');
headers.append('X-Request-ID', Date.now().toString());

fetch('https://api.example.com/data', {
    method: 'GET',
    headers: headers
});

// Или объектом
fetch('https://api.example.com/data', {
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token123'
    }
});
6.2. Чтение заголовков ответа
javascript
const response = await fetch('https://api.github.com/users/octocat');

// Чтение заголовков
console.log('Content-Type:', response.headers.get('Content-Type'));
console.log('Rate Limit:', response.headers.get('X-RateLimit-Limit'));
console.log('Rate Remaining:', response.headers.get('X-RateLimit-Remaining'));

// Перебор всех заголовков
for (let [key, value] of response.headers) {
    console.log(`${key}: ${value}`);
}
6.3. Аутентификация
javascript
// Basic Auth
const username = 'user';
const password = 'pass';
const basicAuth = btoa(`${username}:${password}`);

fetch('https://api.example.com/protected', {
    headers: {
        'Authorization': `Basic ${basicAuth}`
    }
});

// Bearer Token
const token = 'your_jwt_token';
fetch('https://api.example.com/protected', {
    headers: {
        'Authorization': `Bearer ${token}`
    }
});

// API Key
fetch('https://api.example.com/data', {
    headers: {
        'X-API-Key': 'your_api_key'
    }
});
7. Асинхронность: async/await
7.1. Последовательные запросы
javascript
// Запрос пользователя, затем его постов
async function getUserWithPosts(userId) {
    try {
        // Получаем пользователя
        const userResponse = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);
        const user = await userResponse.json();
        
        console.log(`Пользователь: ${user.name}`);
        
        // Получаем посты пользователя
        const postsResponse = await fetch(`https://jsonplaceholder.typicode.com/posts?userId=${userId}`);
        const posts = await postsResponse.json();
        
        console.log(`Постов: ${posts.length}`);
        
        return { user, posts };
        
    } catch (error) {
        console.error('Ошибка:', error);
        throw error;
    }
}

// Использование
getUserWithPosts(1).then(result => {
    console.log('Первый пост:', result.posts[0].title);
});
7.2. Параллельные запросы
javascript
// Параллельное выполнение нескольких запросов
async function loadMultipleData() {
    try {
        // Запускаем запросы параллельно
        const [usersRes, postsRes, commentsRes] = await Promise.all([
            fetch('https://jsonplaceholder.typicode.com/users'),
            fetch('https://jsonplaceholder.typicode.com/posts'),
            fetch('https://jsonplaceholder.typicode.com/comments')
        ]);
        
        // Парсим результаты
        const users = await usersRes.json();
        const posts = await postsRes.json();
        const comments = await commentsRes.json();
        
        console.log(`Пользователей: ${users.length}`);
        console.log(`Постов: ${posts.length}`);
        console.log(`Комментариев: ${comments.length}`);
        
        return { users, posts, comments };
        
    } catch (error) {
        console.error('Ошибка при загрузке:', error);
    }
}

// Promise.allSettled — не ждёт успеха всех запросов
async function loadWithFallback() {
    const results = await Promise.allSettled([
        fetch('https://api1.example.com/data'),
        fetch('https://api2.example.com/data'),
        fetch('https://api3.example.com/data')
    ]);
    
    results.forEach((result, index) => {
        if (result.status === 'fulfilled') {
            console.log(`API ${index + 1} успешно:`, result.value);
        } else {
            console.log(`API ${index + 1} ошибка:`, result.reason);
        }
    });
}
7.3. AbortController (отмена запроса)
javascript
// Отмена запроса при превышении времени или по действию пользователя
function fetchWithTimeout(url, timeout = 5000) {
    const controller = new AbortController();
    const signal = controller.signal;
    
    // Таймаут
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    return fetch(url, { signal })
        .then(response => {
            clearTimeout(timeoutId);
            return response.json();
        })
        .catch(error => {
            if (error.name === 'AbortError') {
                throw new Error('Запрос прерван по таймауту');
            }
            throw error;
        });
}

// Отмена запроса по нажатию кнопки
let currentController = null;

async function searchWithCancel(query) {
    // Отменяем предыдущий запрос
    if (currentController) {
        currentController.abort();
    }
    
    currentController = new AbortController();
    
    try {
        const response = await fetch(`/api/search?q=${query}`, {
            signal: currentController.signal
        });
        
        const results = await response.json();
        displayResults(results);
        
    } catch (error) {
        if (error.name !== 'AbortError') {
            console.error('Ошибка поиска:', error);
        }
    } finally {
        currentController = null;
    }
}
8. Практические примеры
8.1. Список пользователей с динамической загрузкой
html
<!DOCTYPE html>
<html>
<head>
    <title>Пользователи</title>
    <style>
        .user-card {
            border: 1px solid #ddd;
            padding: 10px;
            margin: 10px;
            border-radius: 5px;
        }
        .loading { text-align: center; padding: 20px; }
        .error { color: red; padding: 20px; text-align: center; }
    </style>
</head>
<body>
    <div id="app">
        <h1>Список пользователей</h1>
        <div id="content" class="loading">Загрузка...</div>
    </div>

    <script>
        async function loadUsers() {
            const contentDiv = document.getElementById('content');
            
            try {
                const response = await fetch('https://jsonplaceholder.typicode.com/users');
                
                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}`);
                }
                
                const users = await response.json();
                
                if (users.length === 0) {
                    contentDiv.innerHTML = '<p>Нет пользователей</p>';
                    return;
                }
                
                // Генерация HTML
                const html = users.map(user => `
                    <div class="user-card">
                        <h3>${user.name}</h3>
                        <p><strong>Email:</strong> ${user.email}</p>
                        <p><strong>Телефон:</strong> ${user.phone}</p>
                        <p><strong>Компания:</strong> ${user.company?.name || '—'}</p>
                        <button onclick="loadUserPosts(${user.id})">Посты</button>
                        <div id="posts-${user.id}"></div>
                    </div>
                `).join('');
                
                contentDiv.innerHTML = html;
                
            } catch (error) {
                contentDiv.innerHTML = `<div class="error">Ошибка: ${error.message}</div>`;
            }
        }
        
        async function loadUserPosts(userId) {
            const postsDiv = document.getElementById(`posts-${userId}`);
            postsDiv.innerHTML = '<div class="loading">Загрузка постов...</div>';
            
            try {
                const response = await fetch(`https://jsonplaceholder.typicode.com/posts?userId=${userId}`);
                const posts = await response.json();
                
                const postsHtml = `
                    <div style="margin-top: 10px;">
                        <strong>Посты (${posts.length}):</strong>
                        <ul>
                            ${posts.map(post => `
                                <li>
                                    <strong>${post.title}</strong>
                                    <p>${post.body.substring(0, 100)}...</p>
                                </li>
                            `).join('')}
                        </ul>
                    </div>
                `;
                
                postsDiv.innerHTML = postsHtml;
                
            } catch (error) {
                postsDiv.innerHTML = `<div class="error">Ошибка загрузки постов</div>`;
            }
        }
        
        // Загружаем пользователей при загрузке страницы
        loadUsers();
    </script>
</body>
</html>
8.2. Форма создания поста
html
<!DOCTYPE html>
<html>
<head>
    <title>Создание поста</title>
    <style>
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input, textarea, select { width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 4px; }
        button { padding: 10px 20px; background: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; }
        button:hover { background: #0056b3; }
        .error { color: red; font-size: 12px; margin-top: 5px; }
        .success { color: green; padding: 10px; background: #d4edda; border-radius: 4px; }
    </style>
</head>
<body>
    <div style="max-width: 600px; margin: 50px auto;">
        <h1>Создать пост</h1>
        
        <form id="postForm">
            <div class="form-group">
                <label for="title">Заголовок *</label>
                <input type="text" id="title" name="title" required>
                <div class="error" id="titleError"></div>
            </div>
            
            <div class="form-group">
                <label for="body">Содержание *</label>
                <textarea id="body" name="body" rows="5" required></textarea>
                <div class="error" id="bodyError"></div>
            </div>
            
            <div class="form-group">
                <label for="userId">Автор</label>
                <select id="userId" name="userId">
                    <option value="1">Иван</option>
                    <option value="2">Мария</option>
                    <option value="3">Петр</option>
                </select>
            </div>
            
            <button type="submit">Создать пост</button>
        </form>
        
        <div id="result"></div>
    </div>

    <script>
        const form = document.getElementById('postForm');
        const resultDiv = document.getElementById('result');
        
        form.addEventListener('submit', async (event) => {
            event.preventDefault();
            
            // Очистка ошибок
            document.querySelectorAll('.error').forEach(el => el.textContent = '');
            resultDiv.innerHTML = '';
            
            // Сбор данных
            const title = document.getElementById('title').value.trim();
            const body = document.getElementById('body').value.trim();
            const userId = parseInt(document.getElementById('userId').value);
            
            // Валидация
            let hasError = false;
            
            if (!title) {
                document.getElementById('titleError').textContent = 'Заголовок обязателен';
                hasError = true;
            }
            
            if (!body) {
                document.getElementById('bodyError').textContent = 'Содержание обязательно';
                hasError = true;
            }
            
            if (hasError) return;
            
            // Отправка данных
            try {
                const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        title: title,
                        body: body,
                        userId: userId
                    })
                });
                
                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}`);
                }
                
                const newPost = await response.json();
                
                resultDiv.innerHTML = `
                    <div class="success">
                        ✅ Пост успешно создан!<br>
                        ID: ${newPost.id}<br>
                        Заголовок: ${newPost.title}
                    </div>
                `;
                
                form.reset();
                
            } catch (error) {
                resultDiv.innerHTML = `<div class="error">Ошибка создания поста: ${error.message}</div>`;
            }
        });
    </script>
</body>
</html>
8.3. Полный CRUD пример (Таблица задач)
html
<!DOCTYPE html>
<html>
<head>
    <title>Менеджер задач</title>
    <style>
        * { box-sizing: border-box; }
        body { font-family: Arial, sans-serif; max-width: 800px; margin: 50px auto; padding: 20px; }
        .todo-item { display: flex; align-items: center; padding: 10px; border-bottom: 1px solid #eee; }
        .todo-title { flex: 1; }
        .completed { text-decoration: line-through; color: #888; }
        button { margin-left: 5px; padding: 5px 10px; cursor: pointer; }
        .add-form { display: flex; gap: 10px; margin-bottom: 20px; }
        .add-form input { flex: 1; padding: 10px; border: 1px solid #ddd; border-radius: 4px; }
        .add-form button { margin: 0; }
        .loading { text-align: center; padding: 20px; }
    </style>
</head>
<body>
    <h1>📝 Менеджер задач</h1>
    
    <div class="add-form">
        <input type="text" id="newTitle" placeholder="Новая задача...">
        <button onclick="createTodo()">➕ Добавить</button>
    </div>
    
    <div id="todoList" class="loading">Загрузка...</div>

    <script>
        const API_URL = 'https://jsonplaceholder.typicode.com/todos';
        
        async function loadTodos() {
            const container = document.getElementById('todoList');
            
            try {
                const response = await fetch(`${API_URL}?_limit=10`);
                const todos = await response.json();
                
                if (todos.length === 0) {
                    container.innerHTML = '<p>Нет задач</p>';
                    return;
                }
                
                container.innerHTML = todos.map(todo => `
                    <div class="todo-item" data-id="${todo.id}">
                        <div class="todo-title">
                            <input type="checkbox" 
                                   ${todo.completed ? 'checked' : ''} 
                                   onchange="toggleTodo(${todo.id}, this.checked)">
                            <span class="${todo.completed ? 'completed' : ''}">${todo.title}</span>
                        </div>
                        <button onclick="editTodo(${todo.id})">✏️</button>
                        <button onclick="deleteTodo(${todo.id})">🗑️</button>
                    </div>
                `).join('');
                
            } catch (error) {
                container.innerHTML = `<div class="error">Ошибка загрузки: ${error.message}</div>`;
            }
        }
        
        async function createTodo() {
            const input = document.getElementById('newTitle');
            const title = input.value.trim();
            
            if (!title) {
                alert('Введите название задачи');
                return;
            }
            
            try {
                const response = await fetch(API_URL, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        title: title,
                        completed: false,
                        userId: 1
                    })
                });
                
                if (response.ok) {
                    input.value = '';
                    loadTodos(); // Обновляем список
                } else {
                    alert('Ошибка создания');
                }
                
            } catch (error) {
                alert('Ошибка: ' + error.message);
            }
        }
        
        async function toggleTodo(id, completed) {
            try {
                await fetch(`${API_URL}/${id}`, {
                    method: 'PATCH',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ completed: completed })
                });
                
                loadTodos(); // Обновляем список
                
            } catch (error) {
                alert('Ошибка обновления');
            }
        }
        
        async function editTodo(id) {
            const newTitle = prompt('Введите новое название:');
            if (!newTitle) return;
            
            try {
                await fetch(`${API_URL}/${id}`, {
                    method: 'PATCH',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ title: newTitle })
                });
                
                loadTodos();
                
            } catch (error) {
                alert('Ошибка обновления');
            }
        }
        
        async function deleteTodo(id) {
            if (!confirm('Удалить задачу?')) return;
            
            try {
                await fetch(`${API_URL}/${id}`, { method: 'DELETE' });
                loadTodos();
                
            } catch (error) {
                alert('Ошибка удаления');
            }
        }
        
        // Загружаем задачи при загрузке страницы
        loadTodos();
    </script>
</body>
</html>
9. Контрольные вопросы
Что такое Fetch API и для чего он используется?

Как выполнить GET запрос с параметрами?

Как отправить JSON данные через POST запрос?

Как обработать ошибку 404 при запросе?

В чём разница между response.ok и response.status?

Как загрузить файл на сервер с помощью Fetch?

Что делает метод response.json()?

Как выполнить несколько запросов параллельно?

Как отменить выполняющийся запрос?

Как добавить заголовки авторизации в запрос?

10. Практическое задание
Задание 1 (базовое)
Создайте HTML-страницу, которая загружает и отображает список пользователей из JSONPlaceholder API при нажатии на кнопку.

Задание 2 (среднее)
Разработайте форму для создания нового поста с валидацией полей и отправкой данных на сервер.

Задание 3 (сложное)
Создайте полноценное SPA-приложение для управления задачами (To-Do) с поддержкой:

Загрузки списка задач

Добавления новой задачи

Отметки о выполнении

Редактирования названия

Удаления задачи

11. Шпаргалка
javascript
// === GET ===
fetch(url)
    .then(res => res.json())
    .then(data => console.log(data));

// === GET с параметрами ===
fetch(`${url}?${new URLSearchParams(params)}`)

// === POST JSON ===
fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
});

// === POST FormData ===
fetch(url, {
    method: 'POST',
    body: new FormData(form)
});

// === PUT ===
fetch(`${url}/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(updatedData)
});

// === DELETE ===
fetch(`${url}/${id}`, { method: 'DELETE' });

// === Обработка ответа ===
if (!response.ok) throw new Error(`HTTP ${response.status}`);
const data = await response.json();

// === Параллельные запросы ===
const [res1, res2] = await Promise.all([fetch(url1), fetch(url2)]);

// === Отмена запроса ===
const controller = new AbortController();
fetch(url, { signal: controller.signal });
controller.abort();
Итог лекции
Вы сегодня:

✅ Изучили Fetch API для выполнения HTTP-запросов

✅ Освоили GET, POST, PUT, DELETE методы

✅ Научились обрабатывать ответы и ошибки

✅ Узнали о параллельных запросах и отмене запросов

✅ Создали практические примеры: список пользователей, форма создания, CRUD задачи

Теперь вы можете создавать динамические веб-приложения, взаимодействующие с сервером без перезагрузки страницы!
