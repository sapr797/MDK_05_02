# Тема 2.25. ORM SQLAlchemy: модели, сессии, связи между таблицами

**Цель лекции:**  
Изучить объектно-реляционное отображение (ORM) на примере SQLAlchemy, освоить создание моделей, работу с сессиями, настройку связей между таблицами (один-ко-многим, многие-ко-многим).

> Главная мысль: **ORM превращает строки SQL в объекты Python. Вы работаете с базой данных как с обычными классами, а не с запросами.**

---

## Содержание

1. [Введение в SQLAlchemy](#1-введение-в-sqlalchemy)
2. [Установка и настройка](#2-установка-и-настройка)
3. [Создание моделей](#3-создание-моделей)
4. [Типы данных и колонки](#4-типы-данных-и-колонки)
5. [Сессии и транзакции](#5-сессии-и-транзакции)
6. [CRUD операции через ORM](#6-crud-операции-через-orm)
7. [Связи между таблицами](#7-связи-между-таблицами)
8. [Практические примеры](#8-практические-примеры)
9. [Контрольные вопросы](#9-контрольные-вопросы)
10. [Практическое задание](#10-практическое-задание)
11. [Шпаргалка](#11-шпаргалка)

---

## 1. Введение в SQLAlchemy

### 1.1. Что такое ORM

**ORM (Object-Relational Mapping)** — технология, которая связывает базы данных с концепциями объектно-ориентированных языков.
┌─────────────────────────────────────────────────────────────────┐
│ БЕЗ ORM (SQL запросы) │
├─────────────────────────────────────────────────────────────────┤
│ cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,)) │
│ user = cursor.fetchone() │
│ name = user[1] # доступ по индексу — неудобно │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ С ORM (SQLAlchemy) │
├─────────────────────────────────────────────────────────────────┤
│ user = session.query(User).filter_by(id=user_id).first() │
│ name = user.name # доступ по атрибуту — удобно │
└─────────────────────────────────────────────────────────────────┘

text

### 1.2. SQLAlchemy vs raw SQL

| Характеристика | Raw SQL | SQLAlchemy ORM |
|----------------|---------|----------------|
| **Скорость** | Высокая | Ниже (накладные расходы) |
| **Удобство** | Низкое | Высокое |
| **Безопасность** | Зависит от реализации | Встроенная защита от SQL инъекций |
| **Переносимость** | Низкая (зависит от диалекта) | Высокая (поддержка разных БД) |
| **Сложность** | Низкая | Высокая (кривая обучения) |
| **Поддержка связей** | Ручная | Автоматическая (lazy loading) |

### 1.3. Когда использовать ORM

| Стоит использовать | Не стоит использовать |
|--------------------|----------------------|
| Сложные связи между таблицами | Простые однотабличные запросы |
| Частые изменения схемы | Высоконагруженные системы |
| Командная разработка | Сложные отчёты с агрегацией |
| Быстрое прототипирование | Оптимизация производительности |

---

## 2. Установка и настройка

### 2.1. Установка SQLAlchemy

```bash
# Установка SQLAlchemy
pip install sqlalchemy

# Для работы с SQLite (встроен в Python)
# Для PostgreSQL
pip install psycopg2-binary

# Для MySQL
pip install pymysql

# Для работы с базой в памяти
# SQLite поддерживается из коробки
2.2. Подключение к базе данных
python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# Создание движка (engine) — подключение к БД
# SQLite: файловая БД
engine = create_engine('sqlite:///myapp.db', echo=True)

# SQLite в памяти (для тестов)
# engine = create_engine('sqlite:///:memory:', echo=True)

# PostgreSQL
# engine = create_engine('postgresql://user:password@localhost/mydb')

# MySQL
# engine = create_engine('mysql+pymysql://user:password@localhost/mydb')

# Базовый класс для моделей
Base = declarative_base()

# Фабрика сессий
SessionLocal = sessionmaker(bind=engine)
2.3. Создание таблиц
python
from sqlalchemy import Column, Integer, String, DateTime, Boolean
from datetime import datetime

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(50), nullable=False)
    email = Column(String(100), unique=True, nullable=False)
    age = Column(Integer, default=0)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)

# Создание всех таблиц в базе данных
Base.metadata.create_all(engine)
3. Создание моделей
3.1. Базовый синтаксис модели
python
from sqlalchemy import Column, Integer, String, Float, Boolean, DateTime, Text
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class Product(Base):
    __tablename__ = 'products'
    
    # Колонки
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(100), nullable=False)
    description = Column(Text)
    price = Column(Float, default=0.0)
    in_stock = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
    
    def __repr__(self):
        return f"<Product(id={self.id}, name={self.name}, price={self.price})>"
3.2. Параметры колонок
Параметр	Описание	Пример
primary_key	Первичный ключ	primary_key=True
autoincrement	Автоинкремент	autoincrement=True
nullable	Может быть NULL	nullable=False
default	Значение по умолчанию	default=0
unique	Уникальное значение	unique=True
index	Создать индекс	index=True
onupdate	При обновлении	onupdate=datetime.utcnow
3.3. Типы данных SQLAlchemy
SQLAlchemy тип	Python тип	SQL тип
Integer	int	INTEGER
Float	float	FLOAT
String(length)	str	VARCHAR(length)
Text	str	TEXT
Boolean	bool	BOOLEAN
DateTime	datetime	DATETIME
Date	date	DATE
Time	time	TIME
JSON	dict/list	JSON
LargeBinary	bytes	BLOB
3.4. Полный пример модели
python
from sqlalchemy import Column, Integer, String, Float, Boolean, DateTime, Text, Numeric
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime
import json

Base = declarative_base()

class Order(Base):
    __tablename__ = 'orders'
    
    id = Column(Integer, primary_key=True)
    order_number = Column(String(20), unique=True, nullable=False)
    customer_name = Column(String(100), nullable=False)
    customer_email = Column(String(100), nullable=False)
    total_amount = Column(Numeric(10, 2), default=0.00)
    status = Column(String(20), default='pending')
    items = Column(Text, default='[]')  # JSON строка
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
    
    @property
    def items_list(self):
        """Получение списка товаров из JSON."""
        return json.loads(self.items) if self.items else []
    
    def set_items(self, items_list):
        """Установка списка товаров как JSON."""
        self.items = json.dumps(items_list, ensure_ascii=False)
    
    def __repr__(self):
        return f"<Order(id={self.id}, number={self.order_number}, total={self.total_amount})>"
4. Сессии и транзакции
4.1. Что такое сессия
Сессия — это рабочая область, в которой происходит взаимодействие с базой данных.

python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Создание движка
engine = create_engine('sqlite:///myapp.db')

# Создание фабрики сессий
Session = sessionmaker(bind=engine)

# Создание сессии
session = Session()

# Работа с сессией...

# Закрытие сессии
session.close()
4.2. Использование контекстного менеджера
python
from contextlib import contextmanager

@contextmanager
def get_db():
    """Контекстный менеджер для работы с БД."""
    session = Session()
    try:
        yield session
        session.commit()
    except Exception as e:
        session.rollback()
        raise e
    finally:
        session.close()

# Использование
with get_db() as session:
    user = session.query(User).filter_by(id=1).first()
    print(user.name)
4.3. Транзакции
python
# Автоматическая транзакция (commit при закрытии)
with get_db() as session:
    user1 = User(name="Иван", email="ivan@example.com")
    user2 = User(name="Петр", email="petr@example.com")
    session.add_all([user1, user2])
    # Автоматический commit при выходе из контекста

# Ручное управление транзакцией
session = Session()
try:
    user = User(name="Анна", email="anna@example.com")
    session.add(user)
    session.commit()  # Фиксируем изменения
except Exception as e:
    session.rollback()  # Откат при ошибке
    raise e
finally:
    session.close()
5. CRUD операции через ORM
5.1. CREATE (создание записей)
python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine('sqlite:///myapp.db')
Session = sessionmaker(bind=engine)
session = Session()

# Добавление одной записи
new_user = User(
    name="Иван Петров",
    email="ivan@example.com",
    age=25
)
session.add(new_user)
session.commit()
print(f"Создан пользователь с ID: {new_user.id}")

# Добавление нескольких записей
users = [
    User(name="Мария", email="maria@example.com", age=30),
    User(name="Петр", email="petr@example.com", age=28),
    User(name="Анна", email="anna@example.com", age=22)
]
session.add_all(users)
session.commit()
print(f"Добавлено пользователей: {len(users)}")

# Добавление или обновление (upsert)
user = User(name="Ольга", email="olga@example.com", age=35)
session.merge(user)  # Если email существует — обновит, иначе создаст
session.commit()
5.2. READ (чтение данных)
python
# Получение всех записей
all_users = session.query(User).all()
for user in all_users:
    print(f"{user.id}: {user.name}")

# Получение одной записи по ID
user = session.query(User).get(1)
print(f"Пользователь: {user.name}")

# Получение с фильтрацией
# filter_by (для точных совпадений)
active_users = session.query(User).filter_by(is_active=True).all()

# filter (для сложных условий)
adults = session.query(User).filter(User.age >= 18).all()
young_users = session.query(User).filter(User.age <= 25).all()
specific = session.query(User).filter(
    User.age >= 18,
    User.age <= 30,
    User.is_active == True
).all()

# Получение одной записи (первая)
user = session.query(User).filter_by(email="ivan@example.com").first()

# Получение записи или None (без исключения)
user = session.query(User).filter_by(email="notexists@example.com").one_or_none()

# Сортировка
sorted_users = session.query(User).order_by(User.age.desc()).all()
sorted_users_asc = session.query(User).order_by(User.age).all()

# Ограничение количества
first_5 = session.query(User).limit(5).all()
offset_10 = session.query(User).offset(10).limit(10).all()

# Подсчёт количества
count = session.query(User).filter(User.is_active == True).count()
5.3. UPDATE (обновление записей)
python
# Способ 1: обновление через атрибуты объекта
user = session.query(User).filter_by(id=1).first()
user.name = "Иван Сергеевич"
user.age = 26
session.commit()

# Способ 2: массовое обновление (update)
session.query(User).filter(User.age < 18).update(
    {"is_active": False}
)
session.commit()

# Способ 3: обновление с вычислением
session.query(User).update(
    {User.age: User.age + 1}
)
session.commit()
5.4. DELETE (удаление записей)
python
# Способ 1: удаление объекта
user = session.query(User).filter_by(id=1).first()
session.delete(user)
session.commit()

# Способ 2: массовое удаление
deleted_count = session.query(User).filter(User.age < 18).delete()
print(f"Удалено записей: {deleted_count}")
session.commit()
6. Связи между таблицами
6.1. Связь один-ко-многим (One-to-Many)
python
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

class Author(Base):
    __tablename__ = 'authors'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True)
    
    # Связь с книгами (один автор — много книг)
    books = relationship("Book", back_populates="author", cascade="all, delete-orphan")

class Book(Base):
    __tablename__ = 'books'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    year = Column(Integer)
    author_id = Column(Integer, ForeignKey('authors.id'))
    
    # Связь с автором (много книг — один автор)
    author = relationship("Author", back_populates="books")

# Использование
author = Author(name="Лев Толстой", email="tolstoy@example.com")
author.books = [
    Book(title="Война и мир", year=1869),
    Book(title="Анна Каренина", year=1877),
    Book(title="Воскресение", year=1899)
]

session.add(author)
session.commit()

# Получение книг автора
author = session.query(Author).filter_by(name="Лев Толстой").first()
for book in author.books:
    print(f"{book.title} ({book.year})")
6.2. Связь многие-ко-многим (Many-to-Many)
python
# Таблица-связка (ассоциативная таблица)
student_course = Table(
    'student_course',
    Base.metadata,
    Column('student_id', Integer, ForeignKey('students.id')),
    Column('course_id', Integer, ForeignKey('courses.id'))
)

class Student(Base):
    __tablename__ = 'students'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True)
    
    # Связь с курсами (многие ко многим)
    courses = relationship("Course", secondary=student_course, back_populates="students")

class Course(Base):
    __tablename__ = 'courses'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    credits = Column(Integer, default=3)
    
    # Связь со студентами (многие ко многим)
    students = relationship("Student", secondary=student_course, back_populates="courses")

# Использование
student = Student(name="Иван", email="ivan@example.com")
course1 = Course(title="Python для начинающих", credits=4)
course2 = Course(title="Базы данных", credits=3)

student.courses = [course1, course2]
session.add(student)
session.commit()

# Получение курсов студента
student = session.query(Student).filter_by(name="Иван").first()
for course in student.courses:
    print(f"{course.title} ({course.credits} кредита)")

# Получение студентов курса
course = session.query(Course).filter_by(title="Python для начинающих").first()
for student in course.students:
    print(student.name)
6.3. Связь один-к-одному (One-to-One)
python
class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    
    # Связь с профилем (один к одному)
    profile = relationship("Profile", back_populates="user", uselist=False, cascade="all, delete-orphan")

class Profile(Base):
    __tablename__ = 'profiles'
    
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('users.id'), unique=True)
    bio = Column(Text)
    avatar_url = Column(String(500))
    
    # Связь с пользователем
    user = relationship("User", back_populates="profile")

# Использование
user = User(name="Иван")
user.profile = Profile(bio="Python разработчик", avatar_url="https://...")
session.add(user)
session.commit()
6.4. Параметры relationship
Параметр	Описание
lazy='select'	Загружать при обращении (по умолчанию)
lazy='joined'	Загружать сразу (JOIN запрос)
lazy='subquery'	Загружать через подзапрос
lazy='dynamic'	Возвращать Query объект
cascade="all, delete-orphan"	Каскадное удаление
uselist=False	Связь один-к-одному
7. Практические примеры
7.1. Полный пример: Библиотека
python
from sqlalchemy import create_engine, Column, Integer, String, DateTime, ForeignKey, Text
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship
from datetime import datetime

# Подключение
engine = create_engine('sqlite:///library.db')
Base = declarative_base()
Session = sessionmaker(bind=engine)

# Модели
class Author(Base):
    __tablename__ = 'authors'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    birth_year = Column(Integer)
    books = relationship("Book", back_populates="author", cascade="all, delete-orphan")

class Book(Base):
    __tablename__ = 'books'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    isbn = Column(String(13), unique=True)
    year = Column(Integer)
    author_id = Column(Integer, ForeignKey('authors.id'))
    author = relationship("Author", back_populates="books")
    loans = relationship("Loan", back_populates="book", cascade="all, delete-orphan")

class Reader(Base):
    __tablename__ = 'readers'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True)
    loans = relationship("Loan", back_populates="reader", cascade="all, delete-orphan")

class Loan(Base):
    __tablename__ = 'loans'
    
    id = Column(Integer, primary_key=True)
    book_id = Column(Integer, ForeignKey('books.id'))
    reader_id = Column(Integer, ForeignKey('readers.id'))
    loan_date = Column(DateTime, default=datetime.now)
    return_date = Column(DateTime)
    book = relationship("Book", back_populates="loans")
    reader = relationship("Reader", back_populates="loans")

# Создание таблиц
Base.metadata.create_all(engine)

# Работа с БД
def main():
    session = Session()
    
    # Создание автора
    author = Author(name="Лев Толстой", birth_year=1828)
    session.add(author)
    
    # Создание книг
    books = [
        Book(title="Война и мир", isbn="978-5-17-1234567", year=1869, author=author),
        Book(title="Анна Каренина", isbn="978-5-17-7654321", year=1877, author=author)
    ]
    session.add_all(books)
    
    # Создание читателя
    reader = Reader(name="Иван Петров", email="ivan@example.com")
    session.add(reader)
    
    session.commit()
    
    # Выдача книги
    loan = Loan(book_id=books[0].id, reader_id=reader.id)
    session.add(loan)
    session.commit()
    
    # Запрос: книги автора
    author = session.query(Author).filter_by(name="Лев Толстой").first()
    print(f"Книги {author.name}:")
    for book in author.books:
        print(f"  - {book.title} ({book.year})")
    
    # Запрос: книги на руках у читателя
    reader = session.query(Reader).filter_by(name="Иван Петров").first()
    print(f"\nКниги у {reader.name}:")
    for loan in reader.loans:
        if loan.return_date is None:
            print(f"  - {loan.book.title} (взята {loan.loan_date.date()})")
    
    session.close()

if __name__ == "__main__":
    main()
8. Контрольные вопросы
Что такое ORM и для чего он нужен?

Как создать модель в SQLAlchemy?

В чём разница между filter и filter_by?

Что такое сессия в SQLAlchemy и зачем она нужна?

Как выполнить JOIN запрос через ORM?

Как настроить связь один-ко-многим?

Что такое lazy='joined' и lazy='select'?

Как выполнить массовое обновление записей?

Как обрабатывать ошибки при работе с БД?

В чём преимущества использования ORM перед raw SQL?

9. Практическое задание
Задание 1 (базовое)
Создайте модели User и Post со связью один-ко-многим. Реализуйте функции для создания пользователя, создания поста, получения всех постов пользователя.

Задание 2 (среднее)
Создайте модели для интернет-магазина: Category, Product, Order, OrderItem. Реализуйте функции для добавления товара в корзину и оформления заказа.

Задание 3 (сложное)
Создайте полноценную систему для библиотеки:

Книги, авторы, читатели

Выдача и возврат книг

Отчёты: популярные книги, задолжники

Поиск по автору, названию

10. Шпаргалка
python
# === ПОДКЛЮЧЕНИЕ ===
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine('sqlite:///db.sqlite')
Session = sessionmaker(bind=engine)
session = Session()

# === МОДЕЛЬ ===
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String(50))

# === CREATE ===
user = User(name="Иван")
session.add(user)
session.commit()

# === READ ===
users = session.query(User).all()
user = session.query(User).filter_by(id=1).first()
user = session.query(User).filter(User.id == 1).first()

# === UPDATE ===
user.name = "Петр"
session.commit()

# === DELETE ===
session.delete(user)
session.commit()

# === СВЯЗИ ===
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

class Post(Base):
    __tablename__ = 'posts'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('users.id'))
    user = relationship("User", back_populates="posts")

User.posts = relationship("Post", back_populates="user")
Итог лекции
Вы сегодня:

✅ Изучили основы SQLAlchemy ORM

✅ Научились создавать модели и работать с сессиями

✅ Освоили CRUD операции через ORM

✅ Настроили связи между таблицами (один-ко-многим, многие-ко-многим)

✅ Создали практический пример — систему библиотеки

Теперь вы можете работать с базами данных на уровне объектов Python!
