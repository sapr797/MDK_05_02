# Тема 2.26. Миграции базы данных (Alembic)

**Цель лекции:**  
Изучить инструмент Alembic для управления миграциями базы данных, научиться создавать, применять и откатывать миграции, управлять версиями схемы БД в процессе разработки.

> Главная мысль: **База данных — это часть кода. Её схема должна быть под версионным контролем, как и весь проект. Alembic делает это возможным.**

---

## Содержание

1. [Введение в миграции](#1-введение-в-миграции)
2. [Установка и настройка Alembic](#2-установка-и-настройка-alembic)
3. [Создание миграций](#3-создание-миграций)
4. [Применение и откат миграций](#4-применение-и-откат-миграций)
5. [Автоматическая генерация миграций](#5-автоматическая-генерация-миграций)
6. [Практические примеры](#6-практические-примеры)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Введение в миграции

### 1.1. Что такое миграции базы данных

**Миграции** — это способ версионного контроля схемы базы данных, позволяющий отслеживать и применять изменения структуры БД.
┌─────────────────────────────────────────────────────────────────┐
│ ЭВОЛЮЦИЯ СХЕМЫ БД │
├─────────────────────────────────────────────────────────────────┤
│ Версия 1 Версия 2 Версия 3 Версия 4 │
│ (initial) (add age) (add email) (create posts) │
│ │ │ │ │ │
│ ▼ ▼ ▼ ▼ │
│ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ │
│ │users│ │users│ │users│ │users│ │
│ │id │ │id │ │id │ │id │ │
│ │name │ ──► │name │ ──► │name │ ──► │name │ │
│ └─────┘ │age │ │age │ │age │ │
│ └─────┘ │email│ │email│ │
│ └─────┘ └─────┘ │
│ ┌─────┐ │
│ │posts│ │
│ └─────┘ │
└─────────────────────────────────────────────────────────────────┘

text

### 1.2. Зачем нужны миграции

| Без миграций | С миграциями |
|--------------|--------------|
| Изменения БД вручную (SQL) | Автоматические изменения |
| Нет истории изменений | Полная история версий |
| Трудно синхронизировать команду | Легко делиться изменениями |
| Ошибки при разворачивании | Откат до любой версии |
| Несовместимость сред | Единая схема во всех средах |

### 1.3. Alembic vs другие инструменты

| Инструмент | Плюсы | Минусы |
|------------|-------|--------|
| **Alembic** | Интеграция с SQLAlchemy, мощный | Сложный для новичков |
| **Django Migrations** | Простой, автоматический | Только для Django |
| **Flyway** | Языконезависимый | Требует Java |
| **Liquibase** | Мощный | Сложный |

---

## 2. Установка и настройка Alembic

### 2.1. Установка

```bash
# Установка Alembic
pip install alembic

# Проверка установки
alembic --version
2.2. Инициализация Alembic
bash
# Инициализация в текущем проекте
alembic init alembic

# Инициализация с шаблоном для async
alembic init -t async alembic
Структура после инициализации:

text
my_project/
├── alembic/
│   ├── versions/          # Папка с миграциями
│   ├── env.py             # Конфигурация окружения
│   ├── script.py.mako     # Шаблон миграций
│   └── README
├── alembic.ini            # Основной конфиг
└── models.py              # Ваши модели SQLAlchemy
2.3. Настройка alembic.ini
ini
# alembic.ini

# Путь к папке с миграциями
script_location = alembic

# Формат имени миграции
file_template = %%(year)d_%%(month)_%%(day)_%%(hour)_%%(minute)_%%(second)_%%(slug)s

# Время в UTC
use_utc = true

# URL подключения к БД (можно закомментировать, указать в env.py)
sqlalchemy.url = sqlite:///./myapp.db

# Логирование
[loggers]
keys = root,sqlalchemy,alembic

[handlers]
keys = console

[formatters]
keys = generic

[logger_root]
level = WARN
handlers = console
qualname =

[logger_sqlalchemy]
level = WARN
handlers =
qualname = sqlalchemy.engine

[logger_alembic]
level = INFO
handlers =
qualname = alembic

[handler_console]
class = StreamHandler
args = (sys.stderr,)
level = NOTSET
formatter = generic

[formatter_generic]
format = %(levelname)-5.5s [%(name)s] %(message)s
datefmt = %H:%M:%S
2.4. Настройка env.py
python
# alembic/env.py

import sys
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context

# Загрузка конфигурации логов
config = context.config
fileConfig(config.config_file_name)

# Добавляем путь к моделям
sys.path.append('.')
from models import Base

# Метаданные для автогенерации
target_metadata = Base.metadata

def run_migrations_offline():
    """Запуск миграций в offline режиме (без подключения)"""
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()

def run_migrations_online():
    """Запуск миграций в online режиме (с подключением)"""
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata
        )

        with context.begin_transaction():
            context.run_migrations()

if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
2.5. Пример моделей для миграций
python
# models.py
from sqlalchemy import create_engine, Column, Integer, String, DateTime, Boolean, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True, nullable=False)
    age = Column(Integer, default=0)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    posts = relationship("Post", back_populates="user")

class Post(Base):
    __tablename__ = 'posts'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(String(1000))
    user_id = Column(Integer, ForeignKey('users.id'))
    created_at = Column(DateTime, default=datetime.utcnow)
    
    user = relationship("User", back_populates="posts")
3. Создание миграций
3.1. Создание пустой миграции
bash
# Создание пустой миграции (ручное редактирование)
alembic revision -m "create_users_table"

# Создание с указанием slug
alembic revision -m "add_email_to_users" --slug add_email_to_users
Созданный файл:

python
# alembic/versions/2025_01_15_1430_1234_create_users_table.py

"""create_users_table

Revision ID: 1234567890ab
Revises: 
Create Date: 2025-01-15 14:30:25.123456
"""

from alembic import op
import sqlalchemy as sa

# revision identifiers
revision = '1234567890ab'
down_revision = None
branch_labels = None
depends_on = None

def upgrade():
    pass

def downgrade():
    pass
3.2. Ручное написание миграции
python
"""create_users_table

Revision ID: 1234567890ab
Revises: 
Create Date: 2025-01-15 14:30:25.123456
"""

from alembic import op
import sqlalchemy as sa

revision = '1234567890ab'
down_revision = None
branch_labels = None
depends_on = None

def upgrade():
    # Создание таблицы users
    op.create_table(
        'users',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('name', sa.String(length=100), nullable=False),
        sa.Column('email', sa.String(length=100), nullable=False),
        sa.Column('age', sa.Integer(), nullable=True),
        sa.Column('is_active', sa.Boolean(), nullable=True),
        sa.Column('created_at', sa.DateTime(), nullable=True),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('email')
    )
    
    # Добавление индекса
    op.create_index('idx_users_email', 'users', ['email'])

def downgrade():
    # Удаление индекса
    op.drop_index('idx_users_email', table_name='users')
    # Удаление таблицы
    op.drop_table('users')
3.3. Добавление новой колонки
python
"""add_password_hash_to_users

Revision ID: 23456789abcd
Revises: 1234567890ab
Create Date: 2025-01-15 15:30:25.123456
"""

from alembic import op
import sqlalchemy as sa

revision = '23456789abcd'
down_revision = '1234567890ab'
branch_labels = None
depends_on = None

def upgrade():
    # Добавление колонки (nullable=True для существующих строк)
    op.add_column('users', sa.Column('password_hash', sa.String(length=255), nullable=True))
    
    # Заполнение колонки для существующих записей
    op.execute("UPDATE users SET password_hash = 'temp_hash' WHERE password_hash IS NULL")
    
    # Изменение на NOT NULL
    op.alter_column('users', 'password_hash', nullable=False)

def downgrade():
    op.drop_column('users', 'password_hash')
3.4. Изменение типа колонки
python
"""change_age_type_to_smallint

Revision ID: 34567890bcde
Revises: 23456789abcd
Create Date: 2025-01-15 16:30:25.123456
"""

from alembic import op
import sqlalchemy as sa

revision = '34567890bcde'
down_revision = '23456789abcd'
branch_labels = None
depends_on = None

def upgrade():
    # Изменение типа колонки
    op.alter_column('users', 'age',
        existing_type=sa.Integer(),
        type_=sa.SmallInteger(),
        existing_nullable=True
    )

def downgrade():
    op.alter_column('users', 'age',
        existing_type=sa.SmallInteger(),
        type_=sa.Integer(),
        existing_nullable=True
    )
3.5. Переименование колонки
python
"""rename_name_to_full_name

Revision ID: 45678901cdef
Revises: 34567890bcde
Create Date: 2025-01-15 17:30:25.123456
"""

from alembic import op

revision = '45678901cdef'
down_revision = '34567890bcde'
branch_labels = None
depends_on = None

def upgrade():
    op.alter_column('users', 'name', new_column_name='full_name')

def downgrade():
    op.alter_column('users', 'full_name', new_column_name='name')
3.6. Создание внешнего ключа
python
"""create_posts_table

Revision ID: 56789012def0
Revises: 45678901cdef
Create Date: 2025-01-15 18:30:25.123456
"""

from alembic import op
import sqlalchemy as sa

revision = '56789012def0'
down_revision = '45678901cdef'
branch_labels = None
depends_on = None

def upgrade():
    # Создание таблицы posts
    op.create_table(
        'posts',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('title', sa.String(length=200), nullable=False),
        sa.Column('content', sa.String(length=1000), nullable=True),
        sa.Column('user_id', sa.Integer(), nullable=False),
        sa.Column('created_at', sa.DateTime(), nullable=True),
        sa.ForeignKeyConstraint(['user_id'], ['users.id'], ),
        sa.PrimaryKeyConstraint('id')
    )
    
    # Создание индекса
    op.create_index('idx_posts_user_id', 'posts', ['user_id'])

def downgrade():
    op.drop_index('idx_posts_user_id', table_name='posts')
    op.drop_table('posts')
4. Применение и откат миграций
4.1. Команды Alembic
bash
# Показать текущую версию
alembic current

# Показать историю миграций
alembic history

# Применить все миграции (до последней)
alembic upgrade head

# Применить конкретную миграцию
alembic upgrade 1234567890ab

# Применить +1 миграцию
alembic upgrade +1

# Откат на 1 миграцию
alembic downgrade -1

# Откат до конкретной миграции
alembic downgrade 1234567890ab

# Откат всех миграций
alembic downgrade base

# Обновить до последней миграции (с созданием новой)
alembic revision --autogenerate -m "add_column"

# Синхронизировать метаданные
alembic stamp head
4.2. Статусы миграций
text
┌─────────────────────────────────────────────────────────────────┐
│                     СТАТУСЫ МИГРАЦИЙ                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  base ─────────────────────────────────────────────────────────┤
│    │                                                            │
│    ▼                                                            │
│  1234567890ab (create_users_table) ← head (текущая последняя)   │
│    │                                                            │
│    ▼                                                            │
│  23456789abcd (add_email_index)                                 │
│    │                                                            │
│    ▼                                                            │
│  34567890bcde (add_password_hash) ← current (текущая версия БД) │
│    │                                                            │
│    ▼                                                            │
│  45678901cdef (create_posts_table)                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
4.3. Пример полного цикла
bash
# 1. Просмотр текущего состояния
$ alembic current
Current revision for sqlite:///myapp.db: base

# 2. Создание первой миграции
$ alembic revision --autogenerate -m "initial_migration"
INFO  [alembic] Generating /path/to/alembic/versions/1234567890ab_initial_migration.py

# 3. Применение миграции
$ alembic upgrade head
INFO  [alembic] Running upgrade  -> 1234567890ab, initial_migration

# 4. Проверка
$ alembic current
Current revision for sqlite:///myapp.db: 1234567890ab

# 5. Изменение моделей (добавление колонки)
# редактируем models.py...

# 6. Создание новой миграции
$ alembic revision --autogenerate -m "add_phone_column"
INFO  [alembic] Generating /path/to/alembic/versions/23456789abcd_add_phone_column.py

# 7. Применение
$ alembic upgrade head

# 8. Откат на предыдущую версию
$ alembic downgrade -1
INFO  [alembic] Running downgrade 23456789abcd -> 1234567890ab, add_phone_column

# 9. Возврат обратно
$ alembic upgrade head
5. Автоматическая генерация миграций
5.1. Настройка для автогенерации
python
# alembic/env.py — важно правильно настроить target_metadata

from models import Base

target_metadata = Base.metadata
5.2. Генерация миграции из моделей
bash
# Автоматическая генерация миграции на основе изменений моделей
alembic revision --autogenerate -m "description_of_changes"
5.3. Пример: добавление модели
python
# Добавим новую модель в models.py
class Category(Base):
    __tablename__ = 'categories'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    slug = Column(String(100), unique=True)

# Генерация миграции
alembic revision --autogenerate -m "add_categories_table"
Сгенерированная миграция:

python
"""add_categories_table

Revision ID: 67890123ef01
Revises: 56789012def0
Create Date: 2025-01-15 19:30:25.123456
"""

from alembic import op
import sqlalchemy as sa

revision = '67890123ef01'
down_revision = '56789012def0'
branch_labels = None
depends_on = None

def upgrade():
    op.create_table('categories',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('name', sa.String(length=100), nullable=False),
        sa.Column('slug', sa.String(length=100), nullable=True),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('slug')
    )

def downgrade():
    op.drop_table('categories')
5.4. Что Alembic может автоматически определить
Тип изменения	Автоопределение
Добавление таблицы	✅
Удаление таблицы	✅
Добавление колонки	✅
Удаление колонки	✅
Изменение типа колонки	✅
Добавление индекса	✅
Удаление индекса	✅
Добавление внешнего ключа	✅
Переименование таблицы	❌ (ручное)
Переименование колонки	❌ (ручное)
6. Практические примеры
6.1. Полный рабочий процесс
python
# models.py — файл с моделями SQLAlchemy
from sqlalchemy import create_engine, Column, Integer, String, DateTime, Boolean, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship, sessionmaker
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    username = Column(String(50), unique=True, nullable=False)
    email = Column(String(100), unique=True, nullable=False)
    password_hash = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)

class Post(Base):
    __tablename__ = 'posts'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(String(5000))
    user_id = Column(Integer, ForeignKey('users.id'))
    created_at = Column(DateTime, default=datetime.utcnow)
    
    user = relationship("User", backref="posts")
bash
# Шаг 1: Инициализация Alembic
alembic init alembic

# Шаг 2: Настройка alembic.ini
# Редактируем sqlalchemy.url = sqlite:///./myapp.db

# Шаг 3: Настройка env.py
# Добавляем import models и target_metadata = Base.metadata

# Шаг 4: Создание первой миграции
alembic revision --autogenerate -m "create_users_and_posts_tables"

# Шаг 5: Применение миграции
alembic upgrade head

# Шаг 6: Изменение модели (добавление колонки)
# Добавляем в модель User: last_login = Column(DateTime)

# Шаг 7: Создание миграции изменения
alembic revision --autogenerate -m "add_last_login_to_users"

# Шаг 8: Применение
alembic upgrade head
6.2. Миграция с данными
python
"""update_existing_users_data

Revision ID: 78901234f012
Revises: 67890123ef01
Create Date: 2025-01-15 20:30:25.123456
"""

from alembic import op
import sqlalchemy as sa

revision = '78901234f012'
down_revision = '67890123ef01'
branch_labels = None
depends_on = None

def upgrade():
    # Добавление новой колонки
    op.add_column('users', sa.Column('status', sa.String(20), nullable=True))
    
    # Обновление существующих записей
    conn = op.get_bind()
    conn.execute(
        "UPDATE users SET status = 'active' WHERE status IS NULL"
    )
    
    # Изменение на NOT NULL
    op.alter_column('users', 'status', nullable=False)

def downgrade():
    op.drop_column('users', 'status')
6.3. Управление миграциями в команде
bash
# Разработчик A создаёт новую миграцию
alembic revision --autogenerate -m "add_avatar_to_users"
alembic upgrade head
git add alembic/versions/
git commit -m "Add avatar to users"
git push

# Разработчик B получает изменения
git pull
alembic upgrade head  # Применяет новую миграцию локально
6.4. Docker интеграция
dockerfile
# Dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

# Применение миграций при запуске
CMD alembic upgrade head && python app.py
yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
    depends_on:
      - db
    command: >
      sh -c "alembic upgrade head &&
             python app.py"

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
7. Контрольные вопросы
Что такое миграции базы данных и зачем они нужны?

Как инициализировать Alembic в проекте?

В чём разница между upgrade и downgrade в миграции?

Как создать миграцию вручную? Как автоматически?

Как применить все миграции? Как откатить одну?

Что делает команда alembic current?

Как увидеть историю миграций?

Как изменить тип колонки через миграцию?

Как добавить внешний ключ через миграцию?

Как Alembic определяет изменения между моделями и БД?

8. Практическое задание
Задание 1 (базовое)
Создайте проект с моделями User (id, name, email). Инициализируйте Alembic, создайте миграцию и примените её.

Задание 2 (среднее)
Добавьте в проект модель Post (id, title, content, user_id). Создайте миграцию, примените её, затем добавьте колонку created_at через новую миграцию.

Задание 3 (сложное)
Создайте систему миграций для интернет-магазина:

Версия 1: таблицы products, categories

Версия 2: таблица orders, связь с products

Версия 3: добавление индексов

Версия 4: изменение типа price на DECIMAL(10,2)

9. Шпаргалка
bash
# === ИНИЦИАЛИЗАЦИЯ ===
alembic init alembic

# === СОЗДАНИЕ МИГРАЦИЙ ===
alembic revision -m "message"                      # пустая
alembic revision --autogenerate -m "message"       # автоматическая

# === ПРИМЕНЕНИЕ ===
alembic upgrade head                               # все
alembic upgrade +1                                 # на 1 вперёд
alembic upgrade <revision>                         # до revision

# === ОТКАТ ===
alembic downgrade -1                               # на 1 назад
alembic downgrade base                             # до начальной

# === ИНФОРМАЦИЯ ===
alembic current                                    # текущая версия
alembic history                                    # история

# === СИНХРОНИЗАЦИЯ ===
alembic stamp head                                 # отметить как применённую
python
# === ОПЕРАЦИИ В МИГРАЦИЯХ ===
# Таблицы
op.create_table('users', ...)
op.drop_table('users')

# Колонки
op.add_column('users', sa.Column('age', sa.Integer))
op.drop_column('users', 'age')
op.alter_column('users', 'age', type_=sa.SmallInteger)

# Индексы
op.create_index('idx_name', 'users', ['name'])
op.drop_index('idx_name', table_name='users')

# Внешние ключи
op.create_foreign_key('fk_user_post', 'posts', 'users', ['user_id'], ['id'])
op.drop_constraint('fk_user_post', 'posts', type_='foreignkey')

# Выполнение SQL
op.execute("UPDATE users SET age = 0 WHERE age IS NULL")
Итог лекции
Вы сегодня:

✅ Изучили концепцию миграций базы данных

✅ Настроили Alembic в проекте

✅ Научились создавать, применять и откатывать миграции

✅ Освоили автоматическую генерацию миграций из моделей

✅ Изучили работу с миграциями в команде

Теперь схема вашей базы данных находится под версионным контролем!
