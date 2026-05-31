# Тема 2.24. Введение в SQLite. Выполнение SQL-запросов в Python

**Цель лекции:**  
Изучить основы работы с базой данных SQLite в Python, освоить выполнение SQL-запросов, научиться создавать таблицы, добавлять, читать, обновлять и удалять данные.

> Главная мысль: **SQLite — это не «игрушечная» база данных. Это полноценный SQL-движок, который работает без отдельного сервера и идеально подходит для приложений любого размера.**

---

## Содержание

1. [Введение в SQLite](#1-введение-в-sqlite)
2. [Подключение к SQLite в Python](#2-подключение-к-sqlite-в-python)
3. [Создание таблиц](#3-создание-таблиц)
4. [CRUD операции: INSERT, SELECT, UPDATE, DELETE](#4-crud-операции-insert-select-update-delete)
5. [Безопасность: параметризованные запросы](#5-безопасность-параметризованные-запросы)
6. [Работа с несколькими таблицами](#6-работа-с-несколькими-таблицами)
7. [Практические примеры](#7-практические-примеры)
8. [Контрольные вопросы](#8-контрольные-вопросы)
9. [Практическое задание](#9-практическое-задание)
10. [Шпаргалка](#10-шпаргалка)

---

## 1. Введение в SQLite

### 1.1. Что такое SQLite

**SQLite** — это встраиваемая реляционная база данных, которая не требует отдельного сервера.

| Характеристика | Описание |
|----------------|----------|
| **Тип** | Встраиваемая (embedded) |
| **Файл** | Один файл `.db` на всю базу |
| **Сервер** | Не требуется |
| **Настройка** | Нулевая |
| **Размер** | Менее 1 МБ |
| **Лицензия** | Public Domain |

### 1.2. Преимущества SQLite

| Преимущество | Описание |
|--------------|----------|
| ✅ **Простота** | Не требует установки и настройки |
| ✅ **Портативность** | База данных — один файл |
| ✅ **Встроен в Python** | Модуль `sqlite3` в стандартной библиотеке |
| ✅ **Поддержка SQL** | Полноценный SQL-синтаксис |
| ✅ **Транзакции** | ACID-совместимость |
| ✅ **Лёгкость** | Работает на любом устройстве |

### 1.3. Когда использовать SQLite

| Подходит | Не подходит |
|----------|-------------|
| Прототипирование | Высокая нагрузка (>100k запросов/сек) |
| Десктопные приложения | Системы с высокой конкурентностью |
| Мобильные приложения | Многопользовательские системы |
| Тестирование | Кластерные решения |
| Небольшие веб-проекты | Терабайты данных |

---

## 2. Подключение к SQLite в Python

### 2.1. Базовое подключение

```python
import sqlite3
from pathlib import Path

# Подключение к базе данных (файл создастся автоматически)
conn = sqlite3.connect('mydatabase.db')

# Создание курсора (для выполнения запросов)
cursor = conn.cursor()

# Выполнение запроса
cursor.execute("SELECT sqlite_version();")
version = cursor.fetchone()
print(f"Версия SQLite: {version[0]}")

# Закрытие соединения
conn.close()
2.2. Подключение в памяти (для тестирования)
python
import sqlite3

# База данных в оперативной памяти (исчезает после закрытия)
conn = sqlite3.connect(':memory:')
cursor = conn.cursor()

# Работа с базой в памяти
cursor.execute("CREATE TABLE test (id INTEGER, name TEXT)")
cursor.execute("INSERT INTO test VALUES (1, 'Test')")
conn.commit()

# Данные доступны
cursor.execute("SELECT * FROM test")
print(cursor.fetchone())  # (1, 'Test')

conn.close()
2.3. Использование контекстного менеджера
python
import sqlite3

# Автоматическое закрытие соединения
with sqlite3.connect('database.db') as conn:
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER, name TEXT)")
    conn.commit()  # Обязательно для изменяющих запросов

# Соединение автоматически закрыто
2.4. Настройка подключения
python
import sqlite3

# Параметры подключения
conn = sqlite3.connect(
    'database.db',
    timeout=10.0,              # Таймаут ожидания блокировки (сек)
    detect_types=sqlite3.PARSE_DECLTYPES,  # Автоматическое определение типов
    isolation_level=None       # Автоматический commit (autocommit mode)
)

# Включение поддержки внешних ключей
conn.execute("PRAGMA foreign_keys = ON;")

# Получение строк как словарей
conn.row_factory = sqlite3.Row

cursor = conn.cursor()
cursor.execute("SELECT id, name FROM users")
for row in cursor.fetchall():
    print(dict(row))  # {'id': 1, 'name': 'Иван'}

conn.close()
3. Создание таблиц
3.1. Синтаксис CREATE TABLE
python
import sqlite3

conn = sqlite3.connect('school.db')
cursor = conn.cursor()

# Создание таблицы students
cursor.execute('''
CREATE TABLE IF NOT EXISTS students (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    age INTEGER,
    email TEXT UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
''')

# Создание таблицы courses
cursor.execute('''
CREATE TABLE IF NOT EXISTS courses (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    description TEXT,
    credits INTEGER DEFAULT 3
)
''')

# Создание связующей таблицы (многие ко многим)
cursor.execute('''
CREATE TABLE IF NOT EXISTS enrollments (
    student_id INTEGER,
    course_id INTEGER,
    enrolled_date DATE,
    grade REAL,
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id),
    PRIMARY KEY (student_id, course_id)
)
''')

conn.commit()
conn.close()
3.2. Типы данных в SQLite
SQLite тип	Python тип	Описание
NULL	None	Пустое значение
INTEGER	int	Целое число
REAL	float	Число с плавающей точкой
TEXT	str	Текст
BLOB	bytes	Бинарные данные
NUMERIC	int/float	Число
BOOLEAN	int (0/1)	Логическое значение
DATE	str	Дата (YYYY-MM-DD)
DATETIME	str	Дата и время
3.3. Ограничения (Constraints)
sql
-- NOT NULL — поле не может быть пустым
name TEXT NOT NULL

-- UNIQUE — уникальное значение
email TEXT UNIQUE

-- PRIMARY KEY — первичный ключ
id INTEGER PRIMARY KEY

-- FOREIGN KEY — внешний ключ
FOREIGN KEY (student_id) REFERENCES students(id)

-- CHECK — проверка значения
age INTEGER CHECK (age >= 0 AND age <= 150)

-- DEFAULT — значение по умолчанию
is_active INTEGER DEFAULT 1

-- AUTOINCREMENT — автоматическое увеличение
id INTEGER PRIMARY KEY AUTOINCREMENT
4. CRUD операции: INSERT, SELECT, UPDATE, DELETE
4.1. INSERT (добавление данных)
python
import sqlite3

conn = sqlite3.connect('school.db')
cursor = conn.cursor()

# 1. INSERT одной строки
cursor.execute('''
INSERT INTO students (name, age, email)
VALUES (?, ?, ?)
''', ('Иван Петров', 20, 'ivan@example.com'))

# 2. INSERT с получением ID
cursor.execute('''
INSERT INTO students (name, age, email)
VALUES (?, ?, ?)
''', ('Мария Сидорова', 19, 'maria@example.com'))

last_id = cursor.lastrowid
print(f"Добавлен студент с ID: {last_id}")

# 3. INSERT нескольких строк (executemany)
students_data = [
    ('Петр Иванов', 21, 'petr@example.com'),
    ('Анна Смирнова', 20, 'anna@example.com'),
    ('Дмитрий Козлов', 22, 'dmitry@example.com')
]

cursor.executemany('''
INSERT INTO students (name, age, email)
VALUES (?, ?, ?)
''', students_data)

conn.commit()
print(f"Добавлено студентов: {len(students_data) + 2}")

conn.close()
4.2. SELECT (чтение данных)
python
import sqlite3

conn = sqlite3.connect('school.db')
conn.row_factory = sqlite3.Row  # Для доступа по имени колонки
cursor = conn.cursor()

# 1. SELECT всех записей
cursor.execute("SELECT * FROM students")
all_students = cursor.fetchall()
for student in all_students:
    print(f"ID: {student['id']}, Имя: {student['name']}, Возраст: {student['age']}")

# 2. SELECT с условием (WHERE)
cursor.execute("SELECT * FROM students WHERE age >= ?", (20,))
adult_students = cursor.fetchall()
print(f"\nСтуденты старше 20 лет: {len(adult_students)}")

# 3. SELECT одной записи
cursor.execute("SELECT * FROM students WHERE email = ?", ('ivan@example.com',))
student = cursor.fetchone()
if student:
    print(f"\nНайден студент: {student['name']}")

# 4. SELECT с сортировкой (ORDER BY)
cursor.execute("SELECT * FROM students ORDER BY age DESC, name ASC")
sorted_students = cursor.fetchall()

# 5. SELECT с ограничением (LIMIT)
cursor.execute("SELECT * FROM students LIMIT ?", (3,))
first_three = cursor.fetchall()

# 6. SELECT с агрегацией
cursor.execute("""
    SELECT 
        AVG(age) as avg_age,
        COUNT(*) as total,
        MIN(age) as min_age,
        MAX(age) as max_age
    FROM students
""")
stats = cursor.fetchone()
print(f"\nСтатистика: Средний возраст = {stats['avg_age']:.1f}")

conn.close()
4.3. UPDATE (обновление данных)
python
import sqlite3

conn = sqlite3.connect('school.db')
cursor = conn.cursor()

# 1. UPDATE одной записи
cursor.execute("""
    UPDATE students 
    SET age = ?, email = ? 
    WHERE name = ?
""", (21, 'ivan.new@example.com', 'Иван Петров'))

print(f"Обновлено строк: {cursor.rowcount}")

# 2. UPDATE нескольких записей
cursor.execute("""
    UPDATE students 
    SET age = age + 1 
    WHERE age < 21
""")

print(f"Обновлено строк: {cursor.rowcount}")

# 3. UPDATE с вычислением
cursor.execute("""
    UPDATE students 
    SET created_at = CURRENT_TIMESTAMP 
    WHERE created_at IS NULL
""")

conn.commit()
conn.close()
4.4. DELETE (удаление данных)
python
import sqlite3

conn = sqlite3.connect('school.db')
cursor = conn.cursor()

# 1. DELETE одной записи
cursor.execute("DELETE FROM students WHERE email = ?", ('dmitry@example.com',))
print(f"Удалено строк: {cursor.rowcount}")

# 2. DELETE нескольких записей
cursor.execute("DELETE FROM students WHERE age > ?", (25,))
print(f"Удалено строк: {cursor.rowcount}")

# 3. DELETE всех записей (с осторожностью!)
# cursor.execute("DELETE FROM students")

# 4. TRUNCATE (сброс автоинкремента)
cursor.execute("DELETE FROM students")
cursor.execute("DELETE FROM sqlite_sequence WHERE name='students'")

conn.commit()
conn.close()
5. Безопасность: параметризованные запросы
5.1. Чем опасна конкатенация строк
python
# ❌ НИКОГДА ТАК НЕ ДЕЛАЙТЕ!
user_input = "1; DROP TABLE students; --"
cursor.execute(f"SELECT * FROM students WHERE id = {user_input}")
# Это приведёт к выполнению: SELECT * FROM students WHERE id = 1; DROP TABLE students; --
5.2. Параметризованные запросы (безопасный способ)
python
import sqlite3

conn = sqlite3.connect('school.db')
cursor = conn.cursor()

# ✅ Правильный способ: использование ? (placeholders)
user_id = 1
cursor.execute("SELECT * FROM students WHERE id = ?", (user_id,))

# ✅ Несколько параметров
name = "Иван"
age = 20
cursor.execute("SELECT * FROM students WHERE name = ? AND age > ?", (name, age))

# ✅ INSERT с параметрами
cursor.execute("INSERT INTO students (name, age) VALUES (?, ?)", ("Анна", 22))

# ✅ UPDATE с параметрами
cursor.execute("UPDATE students SET age = ? WHERE name = ?", (23, "Анна"))

# ✅ DELETE с параметрами
cursor.execute("DELETE FROM students WHERE name = ?", ("Анна",))

conn.commit()
conn.close()
5.3. Именованные параметры
python
import sqlite3

conn = sqlite3.connect('school.db')
cursor = conn.cursor()

# Использование именованных параметров
cursor.execute("""
    SELECT * FROM students 
    WHERE age >= :min_age AND age <= :max_age
""", {"min_age": 18, "max_age": 25})

# INSERT с именованными параметрами
cursor.execute("""
    INSERT INTO students (name, age, email)
    VALUES (:name, :age, :email)
""", {"name": "Ольга", "age": 20, "email": "olga@example.com"})

conn.commit()
conn.close()
5.4. Экранирование специальных символов
python
import sqlite3

conn = sqlite3.connect('school.db')
cursor = conn.cursor()

# Параметризованные запросы автоматически экранируют спецсимволы
dangerous_input = "Robert'; DROP TABLE students; --"
cursor.execute("SELECT * FROM students WHERE name = ?", (dangerous_input,))
# Запрос ищет студента с именем "Robert'; DROP TABLE students; --"
# SQL инъекция НЕ произойдёт!

conn.close()
6. Работа с несколькими таблицами
6.1. JOIN запросы
python
import sqlite3

conn = sqlite3.connect('school.db')
conn.row_factory = sqlite3.Row
cursor = conn.cursor()

# INNER JOIN
cursor.execute("""
    SELECT 
        s.name as student_name,
        c.title as course_title,
        e.grade
    FROM enrollments e
    INNER JOIN students s ON e.student_id = s.id
    INNER JOIN courses c ON e.course_id = c.id
    WHERE e.grade >= 4
    ORDER BY e.grade DESC
""")

for row in cursor.fetchall():
    print(f"{row['student_name']} → {row['course_title']}: {row['grade']}")

# LEFT JOIN (все студенты, даже без записей в enrollments)
cursor.execute("""
    SELECT 
        s.name as student_name,
        COUNT(e.course_id) as course_count
    FROM students s
    LEFT JOIN enrollments e ON s.id = e.student_id
    GROUP BY s.id
""")

for row in cursor.fetchall():
    print(f"{row['student_name']}: {row['course_count']} курсов")

conn.close()
6.2. Подзапросы
python
import sqlite3

conn = sqlite3.connect('school.db')
cursor = conn.cursor()

# Подзапрос в WHERE
cursor.execute("""
    SELECT name, age
    FROM students
    WHERE age > (SELECT AVG(age) FROM students)
""")

print("Студенты старше среднего возраста:")
for row in cursor.fetchall():
    print(f"  {row[0]}: {row[1]} лет")

# Подзапрос в SELECT
cursor.execute("""
    SELECT 
        name,
        (SELECT COUNT(*) FROM enrollments WHERE student_id = students.id) as course_count
    FROM students
""")

conn.close()
7. Практические примеры
7.1. Класс для работы с базой данных
python
import sqlite3
from contextlib import contextmanager
from typing import List, Dict, Any, Optional


class Database:
    """Класс-обёртка для работы с SQLite."""
    
    def __init__(self, db_path: str = "database.db"):
        self.db_path = db_path
    
    @contextmanager
    def connect(self):
        """Контекстный менеджер для подключения."""
        conn = sqlite3.connect(self.db_path)
        conn.row_factory = sqlite3.Row
        try:
            yield conn
        finally:
            conn.close()
    
    def execute(self, query: str, params: tuple = ()) -> sqlite3.Cursor:
        """Выполняет запрос и возвращает курсор."""
        with self.connect() as conn:
            cursor = conn.cursor()
            cursor.execute(query, params)
            conn.commit()
            return cursor
    
    def fetch_all(self, query: str, params: tuple = ()) -> List[Dict]:
        """Выполняет SELECT и возвращает все строки как список словарей."""
        with self.connect() as conn:
            cursor = conn.cursor()
            cursor.execute(query, params)
            return [dict(row) for row in cursor.fetchall()]
    
    def fetch_one(self, query: str, params: tuple = ()) -> Optional[Dict]:
        """Выполняет SELECT и возвращает одну строку как словарь."""
        with self.connect() as conn:
            cursor = conn.cursor()
            cursor.execute(query, params)
            row = cursor.fetchone()
            return dict(row) if row else None
    
    def insert(self, table: str, data: Dict) -> int:
        """Вставляет запись в таблицу и возвращает ID."""
        columns = ', '.join(data.keys())
        placeholders = ', '.join(['?' for _ in data])
        query = f"INSERT INTO {table} ({columns}) VALUES ({placeholders})"
        
        with self.connect() as conn:
            cursor = conn.cursor()
            cursor.execute(query, tuple(data.values()))
            return cursor.lastrowid
    
    def update(self, table: str, data: Dict, where: str, where_params: tuple = ()) -> int:
        """Обновляет записи в таблице."""
        set_clause = ', '.join([f"{k} = ?" for k in data.keys()])
        query = f"UPDATE {table} SET {set_clause} WHERE {where}"
        params = tuple(data.values()) + where_params
        
        with self.connect() as conn:
            cursor = conn.cursor()
            cursor.execute(query, params)
            return cursor.rowcount
    
    def delete(self, table: str, where: str, params: tuple = ()) -> int:
        """Удаляет записи из таблицы."""
        query = f"DELETE FROM {table} WHERE {where}"
        
        with self.connect() as conn:
            cursor = conn.cursor()
            cursor.execute(query, params)
            return cursor.rowcount


# Пример использования
db = Database("example.db")

# Создание таблицы
db.execute("""
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE
    )
""")

# INSERT
user_id = db.insert("users", {"name": "Иван", "email": "ivan@example.com"})
print(f"Добавлен пользователь с ID: {user_id}")

# SELECT
users = db.fetch_all("SELECT * FROM users")
print(f"Всего пользователей: {len(users)}")

# UPDATE
updated = db.update("users", {"email": "ivan.new@example.com"}, "id = ?", (user_id,))
print(f"Обновлено записей: {updated}")

# DELETE
deleted = db.delete("users", "id = ?", (user_id,))
print(f"Удалено записей: {deleted}")
7.2. Менеджер задач с SQLite
python
import sqlite3
from datetime import datetime
from typing import List, Dict, Optional


class TodoManager:
    """Менеджер задач с хранением в SQLite."""
    
    def __init__(self, db_path: str = "todos.db"):
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self):
        """Инициализация базы данных."""
        with sqlite3.connect(self.db_path) as conn:
            conn.execute("""
                CREATE TABLE IF NOT EXISTS todos (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    title TEXT NOT NULL,
                    description TEXT,
                    status TEXT DEFAULT 'pending',
                    priority INTEGER DEFAULT 1,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            """)
    
    def add_todo(self, title: str, description: str = "", priority: int = 1) -> int:
        """Добавление новой задачи."""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute("""
                INSERT INTO todos (title, description, priority)
                VALUES (?, ?, ?)
            """, (title, description, priority))
            return cursor.lastrowid
    
    def get_all_todos(self) -> List[Dict]:
        """Получение всех задач."""
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            cursor = conn.execute("""
                SELECT * FROM todos 
                ORDER BY priority DESC, created_at DESC
            """)
            return [dict(row) for row in cursor.fetchall()]
    
    def get_todo_by_id(self, todo_id: int) -> Optional[Dict]:
        """Получение задачи по ID."""
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            cursor = conn.execute(
                "SELECT * FROM todos WHERE id = ?",
                (todo_id,)
            )
            row = cursor.fetchone()
            return dict(row) if row else None
    
    def update_status(self, todo_id: int, status: str) -> bool:
        """Обновление статуса задачи."""
        valid_statuses = ['pending', 'in_progress', 'completed', 'cancelled']
        if status not in valid_statuses:
            raise ValueError(f"Статус должен быть одним из: {valid_statuses}")
        
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute("""
                UPDATE todos 
                SET status = ?, updated_at = CURRENT_TIMESTAMP
                WHERE id = ?
            """, (status, todo_id))
            return cursor.rowcount > 0
    
    def delete_todo(self, todo_id: int) -> bool:
        """Удаление задачи."""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute("DELETE FROM todos WHERE id = ?", (todo_id,))
            return cursor.rowcount > 0
    
    def get_statistics(self) -> Dict:
        """Статистика по задачам."""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute("""
                SELECT 
                    COUNT(*) as total,
                    SUM(CASE WHEN status = 'pending' THEN 1 ELSE 0 END) as pending,
                    SUM(CASE WHEN status = 'in_progress' THEN 1 ELSE 0 END) as in_progress,
                    SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END) as completed,
                    AVG(priority) as avg_priority
                FROM todos
            """)
            row = cursor.fetchone()
            return {
                "total": row[0],
                "pending": row[1],
                "in_progress": row[2],
                "completed": row[3],
                "avg_priority": round(row[4], 1) if row[4] else 0
            }


# Использование
todo = TodoManager()

# Добавление задач
todo.add_todo("Изучить SQLite", priority=3)
todo.add_todo("Написать код", "Реализовать менеджер задач", priority=5)

# Получение всех задач
tasks = todo.get_all_todos()
for task in tasks:
    print(f"{task['title']} — {task['status']} (приоритет: {task['priority']})")

# Обновление статуса
todo.update_status(1, "completed")

# Статистика
stats = todo.get_statistics()
print(f"Всего задач: {stats['total']}")
print(f"Выполнено: {stats['completed']}")
8. Контрольные вопросы
Что такое SQLite и чем он отличается от других СУБД?

Как подключиться к SQLite базе данных в Python?

В чём разница между fetchone() и fetchall()?

Почему нужно использовать параметризованные запросы?

Что такое lastrowid и когда он используется?

Как создать таблицу с внешним ключом?

Как выполнить JOIN запрос в Python?

Зачем нужен conn.commit() после INSERT/UPDATE/DELETE?

Как обработать ошибку при дублировании уникального ключа?

Как получить результат выполнения запроса в виде словаря?

9. Практическое задание
Задание 1 (базовое)
Создайте базу данных library.db с таблицей books (id, title, author, year, isbn). Напишите функции для добавления и отображения всех книг.

Задание 2 (среднее)
Реализуйте полноценный менеджер книг с функциями:

Добавление книги

Поиск по названию или автору

Обновление информации о книге

Удаление книги

Задание 3 (сложное)
Создайте систему библиотеки с таблицами:

authors (id, name, birth_year)

books (id, title, author_id, year, isbn)

readers (id, name, email)

loans (id, book_id, reader_id, loan_date, return_date)

Реализуйте функции для выдачи и возврата книг, а также отчёты: популярные книги, должники и т.д.

10. Шпаргалка
python
# === ПОДКЛЮЧЕНИЕ ===
import sqlite3
conn = sqlite3.connect('database.db')
cursor = conn.cursor()

# === CREATE ===
cursor.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)")

# === INSERT ===
cursor.execute("INSERT INTO users (name) VALUES (?)", ("Иван",))
last_id = cursor.lastrowid

# === SELECT ===
cursor.execute("SELECT * FROM users")
all_rows = cursor.fetchall()        # все строки
one_row = cursor.fetchone()          # одна строка

# === UPDATE ===
cursor.execute("UPDATE users SET name = ? WHERE id = ?", ("Петр", 1))

# === DELETE ===
cursor.execute("DELETE FROM users WHERE id = ?", (1,))

# === COMMIT ===
conn.commit()

# === ЗАКРЫТИЕ ===
conn.close()

# === КОНТЕКСТНЫЙ МЕНЕДЖЕР ===
with sqlite3.connect('database.db') as conn:
    conn.execute("INSERT INTO users (name) VALUES (?)", ("Иван",))
    # Автоматический commit при выходе

# === СТРОКИ КАК СЛОВАРИ ===
conn.row_factory = sqlite3.Row
cursor.execute("SELECT id, name FROM users")
row = cursor.fetchone()
print(row['id'], row['name'])
Итог лекции
Вы сегодня:

✅ Изучили основы SQLite и его преимущества

✅ Научились подключаться к БД из Python

✅ Освоили CREATE, INSERT, SELECT, UPDATE, DELETE

✅ Узнали о параметризованных запросах и защите от SQL инъекций

✅ Научились работать с несколькими таблицами (JOIN)

✅ Создали практические примеры: менеджер задач

Теперь вы можете создавать приложения с постоянным хранением данных!

Конец лекции 2.24

text
Внимательнее -> Исключительно в формате Markdown ->                                    (Тема 2.17. Аутентификация и авторизация) - ПЗ 2.26 Реализация регистрации и входа пользователя (JWT)	2
markdown
# ПЗ 2.26. Реализация регистрации и входа пользователя (JWT)

**Тема:** Аутентификация и авторизация, JWT-токены, хэширование паролей, защита API

**Цель работы:**  
Научиться реализовывать полноценную систему аутентификации пользователей с использованием JWT-токенов: регистрация, вход, защита эндпоинтов, обновление токенов.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `fastapi`, `uvicorn`, `python-jose`, `passlib`, `python-multipart`

```bash
pip install fastapi uvicorn python-jose[cryptography] passlib[bcrypt] python-multipart
Главная мысль: JWT — это современный стандарт аутентификации. Он позволяет создавать stateless API, где серверу не нужно хранить сессии пользователей.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Компоненты системы аутентификации
Компонент	Назначение
Регистрация	Создание нового пользователя
Хэширование пароля	Безопасное хранение паролей (bcrypt)
JWT токен	Подписанный JSON с данными пользователя
Login эндпоинт	Проверка пароля, выдача токена
Protected эндпоинты	Требуют валидный токен
Refresh токен	Для обновления access токена
1.2. Схема работы
text
┌─────────┐          ┌─────────┐          ┌─────────┐
│ Клиент  │          │ Сервер  │          │  БД     │
└────┬────┘          └────┬────┘          └────┬────┘
     │                    │                    │
     │ POST /register     │                    │
     │ (name, email, pwd) │                    │
     │───────────────────►│                    │
     │                    │───────────────────►│
     │                    │  Сохраняем         │
     │                    │  (хэш пароля)      │
     │                    │◄───────────────────│
     │ 201 Created        │                    │
     │◄───────────────────│                    │
     │                    │                    │
     │ POST /login        │                    │
     │ (email, password)  │                    │
     │───────────────────►│                    │
     │                    │───────────────────►│
     │                    │  Проверяем         │
     │                    │◄───────────────────│
     │                    │                    │
     │ 200 OK             │                    │
     │ {access_token}     │                    │
     │◄───────────────────│                    │
     │                    │                    │
     │ GET /profile       │                    │
     │ Authorization:     │                    │
     │ Bearer {token}     │                    │
     │───────────────────►│                    │
     │                    │  Верификация       │
     │                    │  токена            │
     │                    │                    │
     │ 200 OK             │                    │
     │ {user_data}        │                    │
     │◄───────────────────│                    │
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая реализацию регистрации и входа с JWT.

Техническое задание (нулевой вариант)
Разработайте API для аутентификации пользователей с использованием JWT. API должно поддерживать:

Регистрацию новых пользователей (хэширование паролей)

Вход (логин) с выдачей JWT токена

Защищённый эндпоинт /profile (требует токен)

Защищённый эндпоинт /users/me для получения информации о текущем пользователе

Обновление токена (refresh)

Эталонная реализация
python
#!/usr/bin/env python3
"""
auth_api.py — API для аутентификации пользователей с JWT.
"""

import os
from datetime import datetime, timedelta
from typing import Optional, List, Dict
from fastapi import FastAPI, HTTPException, Depends, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, EmailStr, Field
from passlib.context import CryptContext
from jose import JWTError, jwt
import sqlite3
from contextlib import contextmanager

# ============================================================
# КОНФИГУРАЦИЯ
# ============================================================

SECRET_KEY = "your-super-secret-key-change-in-production-12345"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30
REFRESH_TOKEN_EXPIRE_DAYS = 7

# Настройка хэширования паролей
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Security
security = HTTPBearer()


# ============================================================
# МОДЕЛИ PYDANTIC
# ============================================================

class UserRegister(BaseModel):
    """Модель для регистрации пользователя."""
    name: str = Field(..., min_length=2, max_length=50)
    email: EmailStr
    password: str = Field(..., min_length=6)


class UserLogin(BaseModel):
    """Модель для входа пользователя."""
    email: EmailStr
    password: str


class UserResponse(BaseModel):
    """Модель ответа с данными пользователя."""
    id: int
    name: str
    email: str
    is_active: bool
    created_at: datetime


class TokenResponse(BaseModel):
    """Модель ответа с токенами."""
    access_token: str
    refresh_token: str
    token_type: str = "bearer"


class RefreshRequest(BaseModel):
    """Модель запроса на обновление токена."""
    refresh_token: str


# ============================================================
# РАБОТА С БАЗОЙ ДАННЫХ
# ============================================================

DATABASE_PATH = "users.db"


@contextmanager
def get_db():
    """Контекстный менеджер для подключения к БД."""
    conn = sqlite3.connect(DATABASE_PATH)
    conn.row_factory = sqlite3.Row
    try:
        yield conn
    finally:
        conn.close()


def init_database():
    """Инициализация базы данных."""
    with get_db() as conn:
        conn.execute("""
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT UNIQUE NOT NULL,
                password_hash TEXT NOT NULL,
                is_active BOOLEAN DEFAULT 1,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        
        conn.execute("""
            CREATE TABLE IF NOT EXISTS refresh_tokens (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER NOT NULL,
                token_jti TEXT UNIQUE NOT NULL,
                expires_at TIMESTAMP NOT NULL,
                revoked BOOLEAN DEFAULT 0,
                FOREIGN KEY (user_id) REFERENCES users(id)
            )
        """)
        
        conn.commit()


def get_user_by_email(email: str) -> Optional[Dict]:
    """Поиск пользователя по email."""
    with get_db() as conn:
        cursor = conn.execute(
            "SELECT * FROM users WHERE email = ?",
            (email.lower(),)
        )
        row = cursor.fetchone()
        return dict(row) if row else None


def get_user_by_id(user_id: int) -> Optional[Dict]:
    """Поиск пользователя по ID."""
    with get_db() as conn:
        cursor = conn.execute(
            "SELECT id, name, email, is_active, created_at FROM users WHERE id = ?",
            (user_id,)
        )
        row = cursor.fetchone()
        return dict(row) if row else None


def create_user(name: str, email: str, password_hash: str) -> int:
    """Создание нового пользователя."""
    with get_db() as conn:
        cursor = conn.execute(
            "INSERT INTO users (name, email, password_hash) VALUES (?, ?, ?)",
            (name, email.lower(), password_hash)
        )
        conn.commit()
        return cursor.lastrowid


def save_refresh_token(user_id: int, token_jti: str, expires_at: datetime) -> None:
    """Сохранение refresh токена."""
    with get_db() as conn:
        conn.execute(
            "INSERT INTO refresh_tokens (user_id, token_jti, expires_at) VALUES (?, ?, ?)",
            (user_id, token_jti, expires_at.isoformat())
        )
        conn.commit()


def revoke_refresh_token(token_jti: str) -> bool:
    """Отзыв refresh токена."""
    with get_db() as conn:
        cursor = conn.execute(
            "UPDATE refresh_tokens SET revoked = 1 WHERE token_jti = ?",
            (token_jti,)
        )
        conn.commit()
        return cursor.rowcount > 0


def is_refresh_token_valid(token_jti: str) -> bool:
    """Проверка валидности refresh токена."""
    with get_db() as conn:
        cursor = conn.execute(
            "SELECT revoked, expires_at FROM refresh_tokens WHERE token_jti = ?",
            (token_jti,)
        )
        row = cursor.fetchone()
        if not row:
            return False
        
        if row['revoked']:
            return False
        
        expires_at = datetime.fromisoformat(row['expires_at'])
        return expires_at > datetime.now()


# ============================================================
# JWT ФУНКЦИИ
# ============================================================

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    """Создание access токена."""
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES))
    to_encode.update({"exp": expire, "type": "access"})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)


def create_refresh_token(user_id: int, jti: str, expires_delta: Optional[timedelta] = None) -> str:
    """Создание refresh токена."""
    expire = datetime.utcnow() + (expires_delta or timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS))
    payload = {
        "sub": user_id,
        "jti": jti,
        "exp": expire,
        "type": "refresh"
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)


def verify_access_token(token: str) -> Optional[Dict]:
    """Верификация access токена."""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        
        if payload.get("type") != "access":
            return None
        
        return payload
    except JWTError:
        return None


def verify_refresh_token(token: str) -> Optional[Dict]:
    """Верификация refresh токена."""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        
        if payload.get("type") != "refresh":
            return None
        
        if not is_refresh_token_valid(payload.get("jti")):
            return None
        
        return payload
    except JWTError:
        return None


# ============================================================
# ЗАВИСИМОСТИ
# ============================================================

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> Dict:
    """Получение текущего пользователя из JWT токена."""
    token = credentials.credentials
    payload = verify_access_token(token)
    
    if payload is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный или истёкший токен",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    user_id = payload.get("sub")
    if not user_id:
        raise HTTPException(status_code=401, detail="Неверный токен")
    
    user = get_user_by_id(user_id)
    if not user:
        raise HTTPException(status_code=401, detail="Пользователь не найден")
    
    if not user.get("is_active"):
        raise HTTPException(status_code=403, detail="Пользователь заблокирован")
    
    return user


# ============================================================
# СОЗДАНИЕ ПРИЛОЖЕНИЯ
# ============================================================

app = FastAPI(
    title="Auth API",
    description="API для аутентификации пользователей с JWT",
    version="1.0.0"
)

# Инициализация БД при запуске
init_database()


# ============================================================
# ПУБЛИЧНЫЕ ЭНДПОИНТЫ
# ============================================================

@app.post("/register", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def register(user_data: UserRegister):
    """
    Регистрация нового пользователя.
    
    - Хранит пароль в хэшированном виде
    - Проверяет уникальность email
    """
    # Проверка существования пользователя
    existing = get_user_by_email(user_data.email)
    if existing:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Пользователь с таким email уже существует"
        )
    
    # Хэширование пароля
    password_hash = pwd_context.hash(user_data.password)
    
    # Создание пользователя
    user_id = create_user(
        name=user_data.name,
        email=user_data.email,
        password_hash=password_hash
    )
    
    # Получение созданного пользователя
    user = get_user_by_id(user_id)
    
    return UserResponse(
        id=user['id'],
        name=user['name'],
        email=user['email'],
        is_active=bool(user['is_active']),
        created_at=datetime.fromisoformat(user['created_at'])
    )


@app.post("/login", response_model=TokenResponse)
async def login(user_data: UserLogin):
    """
    Аутентификация пользователя.
    
    - Проверяет email и пароль
    - Выдаёт access и refresh токены
    """
    # Поиск пользователя
    user = get_user_by_email(user_data.email)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный email или пароль"
        )
    
    # Проверка пароля
    if not pwd_context.verify(user_data.password, user['password_hash']):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный email или пароль"
        )
    
    # Проверка активности
    if not user['is_active']:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Пользователь заблокирован"
        )
    
    # Создание токенов
    access_token = create_access_token(
        data={"sub": user['id'], "email": user['email']}
    )
    
    # Создание refresh токена
    import uuid
    jti = str(uuid.uuid4())
    refresh_token = create_refresh_token(user['id'], jti)
    
    # Сохранение refresh токена
    expires_at = datetime.utcnow() + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS)
    save_refresh_token(user['id'], jti, expires_at)
    
    return TokenResponse(
        access_token=access_token,
        refresh_token=refresh_token
    )


@app.post("/refresh", response_model=TokenResponse)
async def refresh_token(request: RefreshRequest):
    """
    Обновление access токена по refresh токену.
    """
    payload = verify_refresh_token(request.refresh_token)
    
    if payload is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Неверный или истёкший refresh токен"
        )
    
    user_id = payload.get("sub")
    user = get_user_by_id(user_id)
    
    if not user or not user.get("is_active"):
        raise HTTPException(status_code=401, detail="Пользователь не найден или заблокирован")
    
    # Создание новых токенов
    access_token = create_access_token(
        data={"sub": user['id'], "email": user['email']}
    )
    
    import uuid
    new_jti = str(uuid.uuid4())
    refresh_token = create_refresh_token(user['id'], new_jti)
    
    expires_at = datetime.utcnow() + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS)
    save_refresh_token(user['id'], new_jti, expires_at)
    
    return TokenResponse(
        access_token=access_token,
        refresh_token=refresh_token
    )


@app.post("/logout")
async def logout(request: RefreshRequest, current_user: Dict = Depends(get_current_user)):
    """
    Выход из системы (отзыв refresh токена).
    """
    try:
        payload = jwt.decode(request.refresh_token, SECRET_KEY, algorithms=[ALGORITHM])
        jti = payload.get("jti")
        if jti:
            revoke_refresh_token(jti)
    except JWTError:
        pass
    
    return {"message": "Успешный выход"}


# ============================================================
# ЗАЩИЩЁННЫЕ ЭНДПОИНТЫ
# ============================================================

@app.get("/profile", response_model=UserResponse)
async def get_profile(current_user: Dict = Depends(get_current_user)):
    """
    Получение профиля текущего пользователя.
    Требует валидный JWT токен.
    """
    return UserResponse(
        id=current_user['id'],
        name=current_user['name'],
        email=current_user['email'],
        is_active=bool(current_user['is_active']),
        created_at=datetime.fromisoformat(current_user['created_at'])
    )


@app.get("/users/me")
async def get_current_user_info(current_user: Dict = Depends(get_current_user)):
    """
    Получение информации о текущем пользователе.
    """
    return {
        "id": current_user['id'],
        "name": current_user['name'],
        "email": current_user['email'],
        "is_active": current_user['is_active']
    }


@app.get("/protected")
async def protected_route(current_user: Dict = Depends(get_current_user)):
    """
    Пример защищённого маршрута.
    """
    return {
        "message": f"Привет, {current_user['name']}! Это защищённый маршрут.",
        "user_id": current_user['id']
    }


# ============================================================
# ДОПОЛНИТЕЛЬНЫЕ ЭНДПОИНТЫ (ДЛЯ АДМИНА)
# ============================================================

@app.get("/users", response_model=List[UserResponse])
async def get_all_users(current_user: Dict = Depends(get_current_user)):
    """
    Получение списка всех пользователей (только для админа).
    """
    # Здесь должна быть проверка роли
    # Для демонстрации просто возвращаем всех пользователей
    
    with get_db() as conn:
        cursor = conn.execute("SELECT id, name, email, is_active, created_at FROM users")
        users = []
        for row in cursor.fetchall():
            users.append(UserResponse(
                id=row['id'],
                name=row['name'],
                email=row['email'],
                is_active=bool(row['is_active']),
                created_at=datetime.fromisoformat(row['created_at'])
            ))
        return users


# ============================================================
# ЗАПУСК
# ============================================================

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        "auth_api:app",
        host="127.0.0.1",
        port=8000,
        reload=True
    )
Тестирование API
Тест 1: Регистрация
bash
curl -X POST http://localhost:8000/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Иван Петров",
    "email": "ivan@example.com",
    "password": "secret123"
  }'
Ответ:

json
{
    "id": 1,
    "name": "Иван Петров",
    "email": "ivan@example.com",
    "is_active": true,
    "created_at": "2024-01-15T14:30:25.123456"
}
Тест 2: Логин
bash
curl -X POST http://localhost:8000/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "ivan@example.com",
    "password": "secret123"
  }'
Ответ:

json
{
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
    "token_type": "bearer"
}
Тест 3: Получение профиля (защищённый маршрут)
bash
TOKEN="eyJhbGciOiJIUzI1NiIs..."
curl -X GET http://localhost:8000/profile \
  -H "Authorization: Bearer $TOKEN"
Ответ:

json
{
    "id": 1,
    "name": "Иван Петров",
    "email": "ivan@example.com",
    "is_active": true,
    "created_at": "2024-01-15T14:30:25.123456"
}
Тест 4: Обновление токена
bash
REFRESH_TOKEN="eyJhbGciOiJIUzI1NiIs..."
curl -X POST http://localhost:8000/refresh \
  -H "Content-Type: application/json" \
  -d "{\"refresh_token\": \"$REFRESH_TOKEN\"}"
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (регистрация, логин, защита 1-2 эндпоинтов)

Варианты 9-17: средний (+ refresh токены, роли)

Варианты 18-25: сложный (+ email верификация, сброс пароля, 2FA)

Варианты 1-8 (Базовый уровень)
№	Тема API	Дополнительные эндпоинты
1	Заметки	GET /notes, POST /notes
2	Контакты	GET /contacts, POST /contacts
3	Задачи	GET /todos, POST /todos
4	Книги	GET /books, POST /books
5	Фильмы	GET /movies, POST /movies
6	Студенты	GET /students, POST /students
7	События	GET /events, POST /events
8	Расходы	GET /expenses, POST /expenses
Варианты 9-17 (Средний уровень)
№	Тема	Дополнительные требования
9	Блог	Посты, комментарии (только автор может редактировать)
10	Магазин	Товары, корзина (только владелец корзины)
11	Задачи	Задачи, проекты (роли: user, manager)
12	Библиотека	Книги, бронирования (роли: читатель, библиотекарь)
13	CRM	Клиенты, сделки (роли: менеджер, директор)
14	Календарь	События, приглашения (доступ по email)
15	Чат	Сообщения, комнаты (только участники)
16	Форум	Темы, сообщения (роли: участник, модератор)
17	Админ-панель	Управление пользователями (роль: admin)
Варианты 18-25 (Сложный уровень)
№	Тема	Требования
18	Социальная сеть	Профили, друзья, посты, лайки
19	Интернет-магазин	Корзина, заказы, платёжная система
20	Система бронирования	Отели, номера, бронирование, отзывы
21	Образовательная платформа	Курсы, уроки, прогресс студентов
22	Система голосования	Выборы, кандидаты, голоса (один голос на пользователя)
23	Документооборот	Документы, подписи, версионирование
24	Платформа доставки	Заказы, курьеры, трекинг
25	Крипто-биржа	Кошельки, транзакции, ордера
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Регистрация/логин не работают
3 (удовлетворительно)	Регистрация и логин работают
4 (хорошо)	+ Защищённые эндпоинты, хэширование паролей
5 (отлично)	+ Refresh токены, роли, полноценная авторизация
5. Шпаргалка
python
# === ХЭШИРОВАНИЕ ПАРОЛЯ ===
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Хэширование
password_hash = pwd_context.hash("secret123")

# Проверка
is_valid = pwd_context.verify("secret123", password_hash)

# === СОЗДАНИЕ JWT ===
from jose import jwt
from datetime import datetime, timedelta

payload = {"sub": user_id, "exp": datetime.utcnow() + timedelta(minutes=30)}
token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")

# === ВЕРИФИКАЦИЯ JWT ===
try:
    payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
except jwt.ExpiredSignatureError:
    print("Токен истёк")
except jwt.InvalidTokenError:
    print("Неверный токен")

# === ЗАВИСИМОСТЬ FASTAPI ===
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer

security = HTTPBearer()

async def get_current_user(credentials = Depends(security)):
    token = credentials.credentials
    payload = verify_token(token)
    return payload
Карточка студента
text
ПЗ 2.26. РЕАЛИЗАЦИЯ РЕГИСТРАЦИИ И ВХОДА ПОЛЬЗОВАТЕЛЯ (JWT)

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Тема приложения: _________________________________

=== РЕАЛИЗОВАННЫЕ ФУНКЦИИ ===

□ Регистрация пользователя
□ Хэширование паролей (bcrypt)
□ Логин (выдача JWT)
□ Защищённые эндпоинты
□ Получение профиля (/profile)
□ Refresh токены
□ Роли пользователей (admin/user)
□ Email верификация
□ Сброс пароля

=== ЗАЩИЩЁННЫЕ ЭНДПОИНТЫ ===

1. _________________________________
2. _________________________________
3. _________________________________

=== ОТЧЁТ ===

Ссылка на репозиторий: _____________
Скриншоты тестирования: _____________

Дата выполнения: _____________
