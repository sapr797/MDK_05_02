# ПЗ 2.25. Реализация CRUD-операций через Fetch API

**Тема:** Клиентские запросы (Fetch API), CRUD-операции, асинхронность, работа с JSON

**Цель работы:**  
Научиться реализовывать полноценные CRUD-операции (Create, Read, Update, Delete) на клиентской стороне с использованием Fetch API и отображать данные на странице без перезагрузки.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленный FastAPI и Uvicorn (для имитации сервера)

```bash
pip install fastapi uvicorn
Главная мысль: Современные веб-приложения работают без перезагрузки страницы. Fetch API + CRUD — основа динамического интерфейса.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. CRUD операции и Fetch API
Операция	HTTP метод	Fetch запрос
Create (создание)	POST	fetch(url, {method:'POST', body: JSON.stringify(data)})
Read (чтение)	GET	fetch(url)
Update (обновление)	PUT/PATCH	fetch(url, {method:'PUT', body: JSON.stringify(data)})
Delete (удаление)	DELETE	fetch(url, {method:'DELETE'})
1.2. Жизненный цикл CRUD-приложения
text
┌─────────────────────────────────────────────────────────────────┐
│                     ЗАГРУЗКА СТРАНИЦЫ                            │
│                          GET /items                             │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ОТОБРАЖЕНИЕ СПИСКА                           │
│                 Рендеринг HTML из JSON                          │
└───────────────────────────────┬─────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│   CREATE      │       │    UPDATE     │       │    DELETE     │
│  POST /items  │       │ PUT /items/id │       │DELETE /items/id│
└───────┬───────┘       └───────┬───────┘       └───────┬───────┘
        │                       │                       │
        └───────────────────────┼───────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       ОБНОВЛЕНИЕ UI                             │
│                   Перерисовка списка                            │
└─────────────────────────────────────────────────────────────────┘
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая реализацию CRUD-операций через Fetch API.

Техническое задание (нулевой вариант)
Разработайте одностраничное приложение (SPA) для управления списком задач (To-Do). Приложение должно поддерживать:

Отображение списка задач

Добавление новой задачи

Редактирование названия задачи

Отметку о выполнении (чекбокс)

Удаление задачи

Все операции выполняются без перезагрузки страницы

Backend (FastAPI) для тестирования
python
# server.py — имитация API сервера для тестирования
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import List, Optional
from datetime import datetime

app = FastAPI(title="Todo API")

# Разрешаем CORS для локального тестирования
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# Модели
class TodoCreate(BaseModel):
    title: str
    completed: bool = False

class Todo(TodoCreate):
    id: int
    created_at: datetime

# Хранилище
todos: List[Todo] = []
counter = 0

@app.get("/todos", response_model=List[Todo])
def get_todos():
    return todos

@app.get("/todos/{todo_id}", response_model=Todo)
def get_todo(todo_id: int):
    for todo in todos:
        if todo.id == todo_id:
            return todo
    raise HTTPException(status_code=404, detail="Todo not found")

@app.post("/todos", response_model=Todo, status_code=201)
def create_todo(todo: TodoCreate):
    global counter
    counter += 1
    new_todo = Todo(
        id=counter,
        title=todo.title,
        completed=todo.completed,
        created_at=datetime.now()
    )
    todos.append(new_todo)
    return new_todo

@app.put("/todos/{todo_id}", response_model=Todo)
def update_todo(todo_id: int, todo: TodoCreate):
    for i, existing in enumerate(todos):
        if existing.id == todo_id:
            updated = Todo(
                id=todo_id,
                title=todo.title,
                completed=todo.completed,
                created_at=existing.created_at
            )
            todos[i] = updated
            return updated
    raise HTTPException(status_code=404, detail="Todo not found")

@app.patch("/todos/{todo_id}", response_model=Todo)
def patch_todo(todo_id: int, title: Optional[str] = None, completed: Optional[bool] = None):
    for i, existing in enumerate(todos):
        if existing.id == todo_id:
            updated = Todo(
                id=todo_id,
                title=title if title is not None else existing.title,
                completed=completed if completed is not None else existing.completed,
                created_at=existing.created_at
            )
            todos[i] = updated
            return updated
    raise HTTPException(status_code=404, detail="Todo not found")

@app.delete("/todos/{todo_id}", status_code=204)
def delete_todo(todo_id: int):
    for i, todo in enumerate(todos):
        if todo.id == todo_id:
            todos.pop(i)
            return
    raise HTTPException(status_code=404, detail="Todo not found")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000)
Frontend (HTML + CSS + JavaScript)
html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📝 To-Do менеджер | CRUD с Fetch API</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            max-width: 800px;
            margin: 0 auto;
        }

        /* Карточка приложения */
        .app-card {
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            overflow: hidden;
        }

        /* Заголовок */
        .header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 30px;
            text-align: center;
        }

        .header h1 {
            font-size: 32px;
            margin-bottom: 10px;
        }

        .header p {
            opacity: 0.9;
        }

        /* Статистика */
        .stats {
            display: flex;
            justify-content: space-around;
            background: #f8f9fa;
            padding: 15px;
            border-bottom: 1px solid #eee;
        }

        .stat-item {
            text-align: center;
        }

        .stat-value {
            font-size: 24px;
            font-weight: bold;
            color: #667eea;
        }

        .stat-label {
            font-size: 12px;
            color: #666;
            margin-top: 5px;
        }

        /* Форма добавления */
        .add-form {
            display: flex;
            gap: 10px;
            padding: 20px;
            background: white;
            border-bottom: 1px solid #eee;
        }

        .add-input {
            flex: 1;
            padding: 12px 15px;
            border: 2px solid #e0e0e0;
            border-radius: 10px;
            font-size: 16px;
            transition: border-color 0.3s;
        }

        .add-input:focus {
            outline: none;
            border-color: #667eea;
        }

        .add-btn {
            padding: 12px 25px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            font-size: 16px;
            font-weight: bold;
            transition: transform 0.2s;
        }

        .add-btn:hover {
            transform: translateY(-2px);
        }

        /* Список задач */
        .todo-list {
            padding: 0 20px 20px;
        }

        .todo-item {
            display: flex;
            align-items: center;
            padding: 15px;
            background: #f8f9fa;
            border-radius: 10px;
            margin-bottom: 10px;
            transition: all 0.3s;
            animation: fadeIn 0.3s ease;
        }

        .todo-item:hover {
            background: #e9ecef;
            transform: translateX(5px);
        }

        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(-10px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        /* Чекбокс */
        .todo-checkbox {
            width: 24px;
            height: 24px;
            margin-right: 15px;
            cursor: pointer;
            accent-color: #28a745;
        }

        /* Название задачи */
        .todo-title {
            flex: 1;
            font-size: 16px;
            word-break: break-word;
            cursor: pointer;
            transition: color 0.3s;
        }

        .todo-title.completed {
            text-decoration: line-through;
            color: #aaa;
        }

        .todo-title:hover {
            color: #667eea;
        }

        /* Режим редактирования */
        .edit-input {
            flex: 1;
            padding: 8px 12px;
            border: 2px solid #667eea;
            border-radius: 8px;
            font-size: 16px;
            margin-right: 10px;
        }

        .save-btn {
            padding: 8px 15px;
            background: #28a745;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            margin-right: 5px;
        }

        .cancel-btn {
            padding: 8px 15px;
            background: #6c757d;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
        }

        /* Кнопки действий */
        .todo-actions {
            display: flex;
            gap: 8px;
            margin-left: 10px;
        }

        .edit-btn, .delete-btn {
            padding: 8px 12px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 14px;
            transition: all 0.2s;
        }

        .edit-btn {
            background: #ffc107;
            color: #333;
        }

        .edit-btn:hover {
            background: #e0a800;
        }

        .delete-btn {
            background: #dc3545;
            color: white;
        }

        .delete-btn:hover {
            background: #c82333;
        }

        /* Состояние загрузки */
        .loading {
            text-align: center;
            padding: 40px;
            color: #666;
        }

        /* Пустой список */
        .empty-state {
            text-align: center;
            padding: 60px 20px;
            color: #999;
        }

        .empty-state-icon {
            font-size: 64px;
            margin-bottom: 20px;
        }

        /* Уведомления */
        .toast {
            position: fixed;
            bottom: 20px;
            right: 20px;
            padding: 12px 20px;
            border-radius: 10px;
            color: white;
            font-size: 14px;
            animation: slideIn 0.3s ease;
            z-index: 1000;
        }

        .toast.success {
            background: #28a745;
        }

        .toast.error {
            background: #dc3545;
        }

        @keyframes slideIn {
            from {
                transform: translateX(100%);
                opacity: 0;
            }
            to {
                transform: translateX(0);
                opacity: 1;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="app-card">
            <div class="header">
                <h1>📝 To-Do Менеджер</h1>
                <p>Управляй своими задачами с помощью Fetch API</p>
            </div>

            <div class="stats">
                <div class="stat-item">
                    <div class="stat-value" id="totalCount">0</div>
                    <div class="stat-label">Всего задач</div>
                </div>
                <div class="stat-item">
                    <div class="stat-value" id="completedCount">0</div>
                    <div class="stat-label">Выполнено</div>
                </div>
                <div class="stat-item">
                    <div class="stat-value" id="pendingCount">0</div>
                    <div class="stat-label">Активных</div>
                </div>
            </div>

            <div class="add-form">
                <input type="text" id="newTodoInput" class="add-input" placeholder="Что нужно сделать?" autocomplete="off">
                <button onclick="createTodo()" class="add-btn">➕ Добавить</button>
            </div>

            <div id="todoList" class="todo-list">
                <div class="loading">Загрузка задач...</div>
            </div>
        </div>
    </div>

    <script>
        // API конфигурация
        const API_URL = 'http://localhost:8000/todos';
        
        // Глобальное состояние
        let todos = [];
        
        // ============================================================
        // ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ
        // ============================================================
        
        function showToast(message, type = 'success') {
            const toast = document.createElement('div');
            toast.className = `toast ${type}`;
            toast.textContent = message;
            document.body.appendChild(toast);
            
            setTimeout(() => {
                toast.remove();
            }, 3000);
        }
        
        function updateStats() {
            const total = todos.length;
            const completed = todos.filter(t => t.completed).length;
            const pending = total - completed;
            
            document.getElementById('totalCount').textContent = total;
            document.getElementById('completedCount').textContent = completed;
            document.getElementById('pendingCount').textContent = pending;
        }
        
        // ============================================================
        // CRUD ОПЕРАЦИИ
        // ============================================================
        
        // READ — получение всех задач
        async function loadTodos() {
            const container = document.getElementById('todoList');
            container.innerHTML = '<div class="loading">Загрузка задач...</div>';
            
            try {
                const response = await fetch(API_URL);
                
                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}`);
                }
                
                todos = await response.json();
                renderTodos();
                updateStats();
                
            } catch (error) {
                console.error('Ошибка загрузки:', error);
                container.innerHTML = `
                    <div class="empty-state">
                        <div class="empty-state-icon">⚠️</div>
                        <p>Ошибка загрузки задач</p>
                        <button onclick="loadTodos()" style="margin-top: 10px; padding: 8px 16px;">Повторить</button>
                    </div>
                `;
            }
        }
        
        // CREATE — создание новой задачи
        async function createTodo() {
            const input = document.getElementById('newTodoInput');
            const title = input.value.trim();
            
            if (!title) {
                showToast('Введите название задачи', 'error');
                return;
            }
            
            try {
                const response = await fetch(API_URL, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        title: title,
                        completed: false
                    })
                });
                
                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}`);
                }
                
                const newTodo = await response.json();
                todos.unshift(newTodo); // Добавляем в начало списка
                renderTodos();
                updateStats();
                
                input.value = '';
                showToast('Задача добавлена!', 'success');
                
            } catch (error) {
                console.error('Ошибка создания:', error);
                showToast('Ошибка при добавлении задачи', 'error');
            }
        }
        
        // UPDATE — обновление статуса (completed)
        async function toggleTodoStatus(id, completed) {
            try {
                const response = await fetch(`${API_URL}/${id}`, {
                    method: 'PATCH',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ completed: completed })
                });
                
                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}`);
                }
                
                const updatedTodo = await response.json();
                
                // Обновляем в локальном массиве
                const index = todos.findIndex(t => t.id === id);
                if (index !== -1) {
                    todos[index] = updatedTodo;
                }
                
                renderTodos();
                updateStats();
                showToast(completed ? 'Задача выполнена! 🎉' : 'Задача отмечена как активная', 'success');
                
            } catch (error) {
                console.error('Ошибка обновления:', error);
                showToast('Ошибка при обновлении статуса', 'error');
            }
        }
        
        // UPDATE — обновление названия
        async function updateTodoTitle(id, newTitle) {
            if (!newTitle.trim()) {
                showToast('Название не может быть пустым', 'error');
                return false;
            }
            
            try {
                const response = await fetch(`${API_URL}/${id}`, {
                    method: 'PATCH',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ title: newTitle.trim() })
                });
                
                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}`);
                }
                
                const updatedTodo = await response.json();
                
                const index = todos.findIndex(t => t.id === id);
                if (index !== -1) {
                    todos[index] = updatedTodo;
                }
                
                renderTodos();
                showToast('Название обновлено!', 'success');
                return true;
                
            } catch (error) {
                console.error('Ошибка обновления:', error);
                showToast('Ошибка при обновлении названия', 'error');
                return false;
            }
        }
        
        // DELETE — удаление задачи
        async function deleteTodo(id) {
            if (!confirm('Удалить задачу?')) return;
            
            try {
                const response = await fetch(`${API_URL}/${id}`, {
                    method: 'DELETE'
                });
                
                if (!response.ok && response.status !== 204) {
                    throw new Error(`HTTP ${response.status}`);
                }
                
                // Удаляем из локального массива
                todos = todos.filter(t => t.id !== id);
                
                renderTodos();
                updateStats();
                showToast('Задача удалена', 'success');
                
            } catch (error) {
                console.error('Ошибка удаления:', error);
                showToast('Ошибка при удалении задачи', 'error');
            }
        }
        
        // ============================================================
        // РЕНДЕРИНГ
        // ============================================================
        
        function startEdit(todoId, currentTitle, element) {
            const todoItem = element.closest('.todo-item');
            const titleSpan = todoItem.querySelector('.todo-title');
            const actionsDiv = todoItem.querySelector('.todo-actions');
            
            // Сохраняем оригинальное содержимое
            const originalTitle = currentTitle;
            
            // Создаём форму редактирования
            const editHtml = `
                <input type="text" class="edit-input" value="${escapeHtml(currentTitle)}">
                <button class="save-btn" onclick="saveEdit(${todoId}, this)">💾 Сохранить</button>
                <button class="cancel-btn" onclick="cancelEdit(${todoId}, '${escapeHtml(originalTitle)}', this)">❌ Отмена</button>
            `;
            
            // Заменяем содержимое
            titleSpan.style.display = 'none';
            actionsDiv.style.display = 'none';
            
            const editDiv = document.createElement('div');
            editDiv.className = 'todo-edit';
            editDiv.innerHTML = editHtml;
            todoItem.insertBefore(editDiv, actionsDiv);
            
            // Фокус на инпут
            const input = editDiv.querySelector('.edit-input');
            input.focus();
            input.select();
        }
        
        async function saveEdit(todoId, button) {
            const editDiv = button.closest('.todo-edit');
            const input = editDiv.querySelector('.edit-input');
            const newTitle = input.value.trim();
            
            if (newTitle) {
                await updateTodoTitle(todoId, newTitle);
            }
            
            cancelEdit(todoId, '', button);
        }
        
        function cancelEdit(todoId, originalTitle, button) {
            const editDiv = button.closest('.todo-edit');
            const todoItem = editDiv.closest('.todo-item');
            const titleSpan = todoItem.querySelector('.todo-title');
            const actionsDiv = todoItem.querySelector('.todo-actions');
            
            titleSpan.style.display = 'block';
            actionsDiv.style.display = 'flex';
            editDiv.remove();
        }
        
        function escapeHtml(text) {
            const div = document.createElement('div');
            div.textContent = text;
            return div.innerHTML;
        }
        
        function renderTodos() {
            const container = document.getElementById('todoList');
            
            if (todos.length === 0) {
                container.innerHTML = `
                    <div class="empty-state">
                        <div class="empty-state-icon">📭</div>
                        <p>У вас пока нет задач</p>
                        <p style="font-size: 14px; margin-top: 10px;">Добавьте первую задачу с помощью формы выше</p>
                    </div>
                `;
                return;
            }
            
            // Сортируем: невыполненные сверху, затем по ID
            const sorted = [...todos].sort((a, b) => {
                if (a.completed !== b.completed) {
                    return a.completed ? 1 : -1;
                }
                return b.id - a.id;
            });
            
            container.innerHTML = sorted.map(todo => `
                <div class="todo-item" data-id="${todo.id}">
                    <input type="checkbox" 
                           class="todo-checkbox" 
                           ${todo.completed ? 'checked' : ''} 
                           onchange="toggleTodoStatus(${todo.id}, this.checked)">
                    <span class="todo-title ${todo.completed ? 'completed' : ''}" 
                          ondblclick="startEdit(${todo.id}, '${escapeHtml(todo.title)}', this)">
                        ${escapeHtml(todo.title)}
                    </span>
                    <div class="todo-actions">
                        <button class="edit-btn" onclick="startEdit(${todo.id}, '${escapeHtml(todo.title)}', this)">✏️</button>
                        <button class="delete-btn" onclick="deleteTodo(${todo.id})">🗑️</button>
                    </div>
                </div>
            `).join('');
        }
        
        // ============================================================
        // ИНИЦИАЛИЗАЦИЯ
        // ============================================================
        
        // Обработка Enter в поле ввода
        document.getElementById('newTodoInput').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                createTodo();
            }
        });
        
        // Загрузка задач при старте
        loadTodos();
    </script>
</body>
</html>
Запуск и тестирование
bash
# Терминал 1: запуск бэкенда
python server.py

# Терминал 2: открыть index.html в браузере
# или использовать Live Server в VS Code
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (один ресурс, базовые операции)

Варианты 9-17: средний (несколько полей, фильтрация)

Варианты 18-25: сложный (несколько ресурсов, связи)

Варианты 1-8 (Базовый уровень)
№	Сущность	Поля	CRUD
1	Заметки	title, content	✅
2	Контакты	name, phone	✅
3	Книги	title, author	✅
4	Фильмы	title, year	✅
5	Студенты	name, group	✅
6	Задачи	title, priority	✅
7	Товары	name, price	✅
8	События	title, date	✅
Варианты 9-17 (Средний уровень)
№	Сущность	Дополнительные поля	Особенности
9	Заметки	tags, pinned	Фильтрация по тегам
10	Контакты	email, address, avatar	Поиск
11	Книги	year, genre, rating	Сортировка
12	Задачи	deadline, description	Приоритеты
13	Сотрудники	position, salary, department	Фильтрация
14	Товары	category, stock, description	Категории
15	Посты	content, likes, comments	Лайки
16	Пользователи	email, role, status	Админ-панель
17	Заказы	items, total, status	Корзина
Варианты 18-25 (Сложный уровень)
№	Приложение	Компоненты	Особенности
18	Trello-клон	Доски, списки, карточки	Drag & Drop
19	Блог	Посты, комментарии, авторы	Связанные данные
20	Корзина магазина	Товары, корзина, заказы	Локальное хранение
21	Расписание	События, категории, напоминания	Календарь
22	Чат	Сообщения, пользователи, комнаты	WebSocket
23	Финансы	Транзакции, категории, отчёты	Графики
24	CRM	Клиенты, сделки, контакты	Связи
25	Панель администратора	Пользователи, права, логи	Аутентификация
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	CRUD не работает или приложение не запускается
3 (удовлетворительно)	Работают Read и Create
4 (хорошо)	Работают Read, Create, Delete
5 (отлично)	Полный CRUD + редактирование inline + анимации + уведомления
5. Шпаргалка
5.1. Шаблон Fetch запроса
javascript
// GET
fetch(url)
    .then(res => res.json())
    .then(data => console.log(data));

// POST
fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
});

// PUT
fetch(`${url}/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
});

// DELETE
fetch(`${url}/${id}`, { method: 'DELETE' });
5.2. Обработка ошибок
javascript
try {
    const response = await fetch(url);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    const data = await response.json();
} catch (error) {
    console.error('Ошибка:', error);
}
5.3. Обновление UI после операции
javascript
async function createItem() {
    await fetch(url, { method: 'POST', ... });
    await loadItems(); // Перезагружаем список
}
Карточка студента
text
ПЗ 2.25. РЕАЛИЗАЦИЯ CRUD-ОПЕРАЦИЙ ЧЕРЕЗ FETCH API

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Сущность: _________________________________

=== CRUD ОПЕРАЦИИ ===

□ CREATE (POST) — добавление
□ READ (GET) — отображение списка
□ UPDATE (PUT/PATCH) — редактирование
□ DELETE (DELETE) — удаление

=== ДОПОЛНИТЕЛЬНЫЕ ФУНКЦИИ ===

□ Фильтрация
□ Поиск
□ Сортировка
□ Валидация
□ Уведомления (toasts)
□ Анимации

=== ОТЧЁТ ===

Ссылка на репозиторий: _____________
Скриншоты работы: _____________

Дата выполнения: _____________
