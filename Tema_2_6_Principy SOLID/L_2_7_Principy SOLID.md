# Тема 2.7. Принципы SOLID в Python. Чистая архитектура. Разделение на слои

**Цель лекции:**  
Научиться проектировать поддерживаемый, тестируемый и расширяемый код, используя принципы SOLID и слоистую архитектуру. Понять, как отделить бизнес-логику от внешних зависимостей.

> Главная мысль: **Хорошая архитектура позволяет менять компоненты системы, не ломая всё остальное.**

---

## Часть 1. Введение в архитектуру кода

### 1.1. Почему архитектура важна

**Проблемы плохой архитектуры:**
- "Спагетти-код" — изменения в одном месте ломают всё
- Сложно тестировать (нужна база данных, API, файлы)
- Сложно расширять (новые требования требуют переписывания)
- Код нельзя переиспользовать

**Признаки хорошей архитектуры:**
- Изменение одного модуля не требует изменений в других
- Модули можно тестировать изолированно
- Легко заменить одну реализацию на другую
- Код понятен новому разработчику

### 1.2. Пирамида качества кода
┌─────────────┐
│ Читаемость │
│ (PEP 8) │
┌─┴─────────────┴─┐
│ Тестируемость │
│ (unit tests) │
┌─┴─────────────────┴─┐
│ Расширяемость │
│ (SOLID, паттерны) │
┌─┴───────────────────────┴─┐
│ Независимость от внешних │
│ зависимостей (Clean Arch) │
└─────────────────────────────┘

text

---

## Часть 2. Принципы SOLID

**SOLID** — пять базовых принципов объектно-ориентированного проектирования.

### 2.1. S: Single Responsibility Principle (SRP)

**Принцип единственной ответственности:** У класса должна быть только одна причина для изменения.

#### ❌ Нарушение SRP

```python
class User:
    """Класс делает слишком много: и данные, и сохранение, и отправку."""
    
    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email
    
    def save_to_database(self) -> None:
        """Сохранение в БД — не ответственность класса User"""
        # код сохранения...
    
    def send_email(self, message: str) -> None:
        """Отправка email — не ответственность класса User"""
        # код отправки...
    
    def generate_report(self) -> str:
        """Генерация отчёта — не ответственность класса User"""
        return f"User: {self.name}"
Почему плохо:

Изменение логики сохранения затронет класс User

Изменение формата email затронет класс User

Трудно тестировать (нужна БД, SMTP сервер)

✅ Соблюдение SRP
python
# data/user.py — только данные и бизнес-логика о пользователе
class User:
    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email
    
    def change_email(self, new_email: str) -> None:
        """Бизнес-правило: email должен содержать @"""
        if '@' not in new_email:
            raise ValueError("Invalid email")
        self.email = new_email


# repositories/user_repo.py — отвечает за сохранение
class UserRepository:
    def save(self, user: User) -> None:
        # код сохранения в БД
        pass


# services/email_service.py — отвечает за отправку email
class EmailService:
    def send_welcome_email(self, user: User) -> None:
        # код отправки email
        pass


# services/report_service.py — отвечает за отчёты
class ReportService:
    def generate_user_report(self, user: User) -> str:
        return f"User report: {user.name}"
2.2. O: Open/Closed Principle (OCP)
Принцип открытости/закрытости: Классы должны быть открыты для расширения, но закрыты для модификации.

❌ Нарушение OCP
python
class DiscountCalculator:
    """При добавлении нового типа скидки нужно менять метод calculate."""
    
    def calculate(self, price: float, discount_type: str) -> float:
        if discount_type == "percentage":
            return price * 0.9
        elif discount_type == "fixed":
            return price - 100
        elif discount_type == "summer_sale":
            return price * 0.8
        # Каждый раз добавляем elif → нарушение OCP
✅ Соблюдение OCP (через наследование)
python
from abc import ABC, abstractmethod

class DiscountStrategy(ABC):
    """Абстрактный класс для стратегий скидок."""
    
    @abstractmethod
    def apply(self, price: float) -> float:
        pass


class PercentageDiscount(DiscountStrategy):
    def __init__(self, percent: float):
        self.percent = percent
    
    def apply(self, price: float) -> float:
        return price * (1 - self.percent / 100)


class FixedDiscount(DiscountStrategy):
    def __init__(self, amount: float):
        self.amount = amount
    
    def apply(self, price: float) -> float:
        return max(0, price - self.amount)


class SummerSaleDiscount(DiscountStrategy):
    def apply(self, price: float) -> float:
        return price * 0.8


class DiscountCalculator:
    """Не требует изменений при добавлении новых скидок."""
    
    def calculate(self, price: float, strategy: DiscountStrategy) -> float:
        return strategy.apply(price)


# Использование
calculator = DiscountCalculator()
price = 1000

strategies = [
    PercentageDiscount(10),
    FixedDiscount(100),
    SummerSaleDiscount()
]

for strategy in strategies:
    print(f"{strategy.__class__.__name__}: {calculator.calculate(price, strategy)}")
2.3. L: Liskov Substitution Principle (LSP)
Принцип подстановки Барбары Лисков: Объекты подкласса должны заменять объекты суперкласса без изменения логики программы.

❌ Нарушение LSP
python
class Bird:
    def fly(self) -> str:
        return "Flying..."
    
    def walk(self) -> str:
        return "Walking..."

class Penguin(Bird):
    def fly(self) -> str:
        raise NotImplementedError("Penguins can't fly!")
    
    def walk(self) -> str:
        return "Waddling..."

def make_bird_fly(bird: Bird) -> None:
    print(bird.fly())  # Упадёт для Penguin!


bird = Bird()
penguin = Penguin()
make_bird_fly(bird)    # OK
make_bird_fly(penguin) # Ошибка! Нарушение LSP
✅ Соблюдение LSP
python
from abc import ABC, abstractmethod

class Bird(ABC):
    @abstractmethod
    def walk(self) -> str:
        pass


class FlyingBird(Bird):
    @abstractmethod
    def fly(self) -> str:
        pass


class Sparrow(FlyingBird):
    def fly(self) -> str:
        return "Sparrow flying..."
    
    def walk(self) -> str:
        return "Sparrow hopping..."


class Penguin(Bird):
    def walk(self) -> str:
        return "Penguin waddling..."


def make_bird_move(bird: Bird) -> None:
    print(bird.walk())


def make_flying_bird_fly(bird: FlyingBird) -> None:
    print(bird.fly())


sparrow = Sparrow()
penguin = Penguin()

make_bird_move(sparrow)
make_bird_move(penguin)
make_flying_bird_fly(sparrow)
# make_flying_bird_fly(penguin)  # Ошибка компиляции! Penguin не FlyingBird
2.4. I: Interface Segregation Principle (ISP)
Принцип разделения интерфейсов: Не следует заставлять классы реализовывать методы, которые они не используют.

❌ Нарушение ISP
python
from abc import ABC, abstractmethod

class Worker(ABC):
    @abstractmethod
    def work(self) -> None:
        pass
    
    @abstractmethod
    def eat(self) -> None:
        pass
    
    @abstractmethod
    def sleep(self) -> None:
        pass
    
    @abstractmethod
    def code(self) -> None:
        pass
    
    @abstractmethod
    def design(self) -> None:
        pass


class Robot(Worker):
    """Роботу нужны не все методы."""
    def work(self): pass
    def eat(self): raise NotImplementedError("Robot doesn't eat")
    def sleep(self): raise NotImplementedError("Robot doesn't sleep")
    def code(self): pass
    def design(self): pass  # Робот не дизайнер!
✅ Соблюдение ISP
python
from abc import ABC, abstractmethod

# Маленькие, специализированные интерфейсы
class Workable(ABC):
    @abstractmethod
    def work(self) -> None:
        pass


class Eatable(ABC):
    @abstractmethod
    def eat(self) -> None:
        pass


class Sleepable(ABC):
    @abstractmethod
    def sleep(self) -> None:
        pass


class Codable(ABC):
    @abstractmethod
    def code(self) -> None:
        pass


class Designable(ABC):
    @abstractmethod
    def design(self) -> None:
        pass


# Классы реализуют только нужные интерфейсы
class Human(Workable, Eatable, Sleepable, Codable, Designable):
    def work(self): pass
    def eat(self): pass
    def sleep(self): pass
    def code(self): pass
    def design(self): pass


class Developer(Workable, Codable, Eatable, Sleepable):
    def work(self): pass
    def code(self): pass
    def eat(self): pass
    def sleep(self): pass


class Robot(Workable, Codable):
    def work(self): pass
    def code(self): pass
2.5. D: Dependency Inversion Principle (DIP)
Принцип инверсии зависимостей:

Модули высокого уровня не зависят от модулей низкого уровня

Оба зависят от абстракций

Абстракции не зависят от деталей, детали зависят от абстракций

❌ Нарушение DIP
python
class MySQLDatabase:
    def connect(self) -> None:
        print("Connecting to MySQL...")
    
    def query(self, sql: str) -> list:
        print(f"Executing SQL: {sql}")
        return []


class UserService:
    """Зависит от конкретной БД, а не от абстракции."""
    
    def __init__(self):
        self.db = MySQLDatabase()  # Прямая зависимость
    
    def get_users(self) -> list:
        self.db.connect()
        return self.db.query("SELECT * FROM users")


# Проблема: нельзя заменить MySQL на PostgreSQL
✅ Соблюдение DIP
python
from abc import ABC, abstractmethod
from typing import List

# Абстракция (интерфейс) для базы данных
class Database(ABC):
    @abstractmethod
    def connect(self) -> None:
        pass
    
    @abstractmethod
    def query(self, sql: str) -> List[dict]:
        pass


# Конкретная реализация для MySQL
class MySQLDatabase(Database):
    def connect(self) -> None:
        print("Connecting to MySQL...")
    
    def query(self, sql: str) -> List[dict]:
        print(f"MySQL: {sql}")
        return []


# Конкретная реализация для PostgreSQL
class PostgreSQLDatabase(Database):
    def connect(self) -> None:
        print("Connecting to PostgreSQL...")
    
    def query(self, sql: str) -> List[dict]:
        print(f"PostgreSQL: {sql}")
        return []


# Конкретная реализация для MongoDB
class MongoDatabase(Database):
    def connect(self) -> None:
        print("Connecting to MongoDB...")
    
    def query(self, sql: str) -> List[dict]:
        print(f"MongoDB: {sql}")
        return []


# Сервис зависит от абстракции, а не от конкретной реализации
class UserService:
    """Зависит от абстракции Database."""
    
    def __init__(self, database: Database):
        self.db = database  # Инъекция зависимости
    
    def get_users(self) -> List[dict]:
        self.db.connect()
        return self.db.query("SELECT * FROM users")


# Использование (Dependency Injection)
mysql_db = MySQLDatabase()
postgres_db = PostgreSQLDatabase()
mongo_db = MongoDatabase()

user_service = UserService(mysql_db)
users = user_service.get_users()  # Использует MySQL

user_service = UserService(postgres_db)
users = user_service.get_users()  # Использует PostgreSQL
Часть 3. Чистая архитектура (Clean Architecture)
3.1. Что такое Clean Architecture
Чистая архитектура (Robert C. Martin) — это способ организации кода, при котором бизнес-логика не зависит от внешних деталей (БД, API, UI, фреймворков).

3.2. Диаграмма слоёв
text
                    ┌─────────────────────────────────────────────┐
                    │                 Внешние слои                  │
                    │  (могут меняться без изменения бизнес-логики)  │
                    │                                               │
                    │  ┌─────────────┐    ┌─────────────────────┐   │
                    │  │   Frameworks│    │        Drivers       │   │
                    │  │  & Drivers  │    │   (Web, DB, API)     │   │
                    │  └──────┬──────┘    └──────────┬──────────┘   │
                    │         │                      │              │
                    │         └──────────────────────┘              │
                    │                       │                        │
                    │  ┌────────────────────┴────────────────────┐  │
                    │  │          Interface Adapters              │  │
                    │  │    (Controllers, Presenters, Gateways)   │  │
                    │  └────────────────────┬────────────────────┘  │
                    │                       │                        │
                    │  ┌────────────────────┴────────────────────┐  │
                    │  │            Use Cases (Services)          │  │
                    │  │         (Бизнес-логика приложения)        │  │
                    │  └────────────────────┬────────────────────┘  │
                    │                       │                        │
                    │  ┌────────────────────┴────────────────────┐  │
                    │  │              Entities (Models)           │  │
                    │  │   (Ключевые бизнес-объекты и правила)     │  │
                    │  └──────────────────────────────────────────┘  │
                    │                                               │
                    │           Внутренние слои (ядро)              │
                    │      (никогда не зависят от внешних)          │
                    └───────────────────────────────────────────────┘
3.3. Правило зависимости
Главное правило чистой архитектуры: зависимости направлены только внутрь. Внешние слои зависят от внутренних, но не наоборот.

text
Entities (ядро) ← Use Cases (сервисы) ← Interface Adapters ← Frameworks
     ↓                  ↓                      ↓                  ↓
  нет зависимостей    ← только от Entities   ← от Use Cases    ← от всех
3.4. Пример: чистая архитектура на Python
Слой 1: Entities (Модели)
python
# src/domain/entities/user.py
from dataclasses import dataclass
from typing import Optional
from enum import Enum


class UserRole(Enum):
    ADMIN = "admin"
    USER = "user"
    GUEST = "guest"


@dataclass
class User:
    """Бизнес-сущность. Не зависит ни от чего."""
    id: Optional[int]
    name: str
    email: str
    role: UserRole
    
    def change_email(self, new_email: str) -> None:
        """Бизнес-правило: email должен содержать @."""
        if '@' not in new_email:
            raise ValueError("Invalid email format")
        self.email = new_email
    
    def is_admin(self) -> bool:
        return self.role == UserRole.ADMIN
    
    def can_delete_user(self, target: 'User') -> bool:
        """Бизнес-правило: админ может удалить любого, кроме себя."""
        if not self.is_admin():
            return False
        return self.id != target.id
Слой 2: Use Cases (Интерфейсы и сервисы)
python
# src/domain/interfaces/user_repository.py
from abc import ABC, abstractmethod
from typing import List, Optional
from src.domain.entities.user import User


class UserRepository(ABC):
    """Абстракция для хранения пользователей."""
    
    @abstractmethod
    def save(self, user: User) -> User:
        pass
    
    @abstractmethod
    def find_by_id(self, user_id: int) -> Optional[User]:
        pass
    
    @abstractmethod
    def find_by_email(self, email: str) -> Optional[User]:
        pass
    
    @abstractmethod
    def delete(self, user_id: int) -> bool:
        pass
    
    @abstractmethod
    def get_all(self) -> List[User]:
        pass


# src/domain/interfaces/email_service.py
class EmailService(ABC):
    @abstractmethod
    def send_welcome_email(self, user: User) -> None:
        pass
python
# src/application/services/user_service.py
from typing import List, Optional
from src.domain.entities.user import User, UserRole
from src.domain.interfaces.user_repository import UserRepository
from src.domain.interfaces.email_service import EmailService


class UserService:
    """
    Use Case (сервис приложения).
    Содержит бизнес-логику приложения.
    Зависит от абстракций, а не от конкретных реализаций.
    """
    
    def __init__(
        self, 
        user_repository: UserRepository,
        email_service: EmailService
    ):
        self.user_repo = user_repository
        self.email_service = email_service
    
    def register_user(self, name: str, email: str) -> User:
        """Регистрация нового пользователя."""
        # Проверка: email не занят
        existing = self.user_repo.find_by_email(email)
        if existing:
            raise ValueError(f"User with email {email} already exists")
        
        # Создаём нового пользователя
        user = User(
            id=None,
            name=name,
            email=email,
            role=UserRole.USER
        )
        
        # Сохраняем
        saved_user = self.user_repo.save(user)
        
        # Отправляем приветственное письмо
        self.email_service.send_welcome_email(saved_user)
        
        return saved_user
    
    def get_user_profile(self, user_id: int) -> Optional[User]:
        """Получение профиля пользователя."""
        user = self.user_repo.find_by_id(user_id)
        if not user:
            return None
        return user
    
    def delete_user(self, admin_user: User, user_id: int) -> bool:
        """Удаление пользователя (только для админа)."""
        target_user = self.user_repo.find_by_id(user_id)
        if not target_user:
            return False
        
        if not admin_user.can_delete_user(target_user):
            raise PermissionError("Not allowed to delete this user")
        
        return self.user_repo.delete(user_id)
Слой 3: Interface Adapters (Адаптеры)
python
# src/infrastructure/repositories/sqlite_user_repository.py
import sqlite3
from typing import List, Optional
from src.domain.entities.user import User, UserRole
from src.domain.interfaces.user_repository import UserRepository


class SQLiteUserRepository(UserRepository):
    """Реализация репозитория для SQLite."""
    
    def __init__(self, db_path: str):
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self) -> None:
        with sqlite3.connect(self.db_path) as conn:
            conn.execute("""
                CREATE TABLE IF NOT EXISTS users (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    name TEXT NOT NULL,
                    email TEXT UNIQUE NOT NULL,
                    role TEXT NOT NULL
                )
            """)
    
    def save(self, user: User) -> User:
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                "INSERT INTO users (name, email, role) VALUES (?, ?, ?)",
                (user.name, user.email, user.role.value)
            )
            user.id = cursor.lastrowid
        return user
    
    def find_by_id(self, user_id: int) -> Optional[User]:
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                "SELECT id, name, email, role FROM users WHERE id = ?",
                (user_id,)
            )
            row = cursor.fetchone()
            if row:
                return User(
                    id=row[0],
                    name=row[1],
                    email=row[2],
                    role=UserRole(row[3])
                )
        return None
    
    def find_by_email(self, email: str) -> Optional[User]:
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                "SELECT id, name, email, role FROM users WHERE email = ?",
                (email,)
            )
            row = cursor.fetchone()
            if row:
                return User(
                    id=row[0],
                    name=row[1],
                    email=row[2],
                    role=UserRole(row[3])
                )
        return None
    
    def delete(self, user_id: int) -> bool:
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute("DELETE FROM users WHERE id = ?", (user_id,))
            return cursor.rowcount > 0
    
    def get_all(self) -> List[User]:
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute("SELECT id, name, email, role FROM users")
            return [
                User(id=row[0], name=row[1], email=row[2], role=UserRole(row[3]))
                for row in cursor.fetchall()
            ]
python
# src/infrastructure/services/smtp_email_service.py
from src.domain.entities.user import User
from src.domain.interfaces.email_service import EmailService


class SMTPEmailService(EmailService):
    """Реализация отправки email через SMTP."""
    
    def __init__(self, smtp_host: str, smtp_port: int):
        self.smtp_host = smtp_host
        self.smtp_port = smtp_port
    
    def send_welcome_email(self, user: User) -> None:
        # Реальная отправка email
        print(f"Sending welcome email to {user.email} via SMTP")
        # ... код отправки
Слой 4: Frameworks (Входные точки)
python
# main.py
from src.application.services.user_service import UserService
from src.infrastructure.repositories.sqlite_user_repository import SQLiteUserRepository
from src.infrastructure.services.smtp_email_service import SMTPEmailService


# Конфигурация (Dependency Injection)
def main():
    # Создаём зависимости
    user_repo = SQLiteUserRepository("database.db")
    email_service = SMTPEmailService("smtp.gmail.com", 587)
    
    # Внедряем зависимости в сервис
    user_service = UserService(user_repo, email_service)
    
    # Используем сервис
    try:
        user = user_service.register_user("Alice", "alice@example.com")
        print(f"User registered: {user}")
        
        profile = user_service.get_user_profile(user.id)
        print(f"Profile: {profile}")
    except ValueError as e:
        print(f"Error: {e}")


# web/api.py — для веб-приложения
from flask import Flask, request, jsonify
from src.application.services.user_service import UserService

def create_app(user_service: UserService) -> Flask:
    app = Flask(__name__)
    
    @app.route('/users', methods=['POST'])
    def create_user():
        data = request.json
        try:
            user = user_service.register_user(data['name'], data['email'])
            return jsonify({'id': user.id, 'name': user.name}), 201
        except ValueError as e:
            return jsonify({'error': str(e)}), 400
    
    return app
Часть 4. Разделение на слои на практике
4.1. Типовая структура проекта по Clean Architecture
text
my_project/
│
├── src/
│   ├── domain/                      # Слой ядра (не зависит от внешнего мира)
│   │   ├── entities/                # Бизнес-сущности
│   │   │   ├── user.py
│   │   │   └── product.py
│   │   └── interfaces/              # Абстракции (порты)
│   │       ├── user_repository.py
│   │       └── email_service.py
│   │
│   ├── application/                 # Слой приложения (use cases)
│   │   ├── services/                # Сервисы с бизнес-логикой приложения
│   │   │   ├── user_service.py
│   │   │   └── auth_service.py
│   │   ├── dto/                     # Data Transfer Objects
│   │   │   └── user_dto.py
│   │   └── exceptions/              # Бизнес-исключения
│   │       └── domain_exceptions.py
│   │
│   ├── infrastructure/              # Слой инфраструктуры (реализации)
│   │   ├── repositories/            # Реализации репозиториев
│   │   │   ├── sqlite_user_repository.py
│   │   │   └── postgres_user_repository.py
│   │   ├── services/                # Внешние сервисы
│   │   │   ├── smtp_email_service.py
│   │   │   └── sendgrid_email_service.py
│   │   └── config/                  # Конфигурация
│   │       └── settings.py
│   │
│   └── interfaces/                  # Пользовательские интерфейсы
│       ├── web/                     # Веб-адаптеры
│       │   ├── api.py
│       │   └── controllers/
│       ├── cli/                     # CLI-адаптеры
│       │   └── commands.py
│       └── presenters/              # Форматирование вывода
│           └── user_presenter.py
│
├── tests/
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── interfaces/
│
├── requirements.txt
└── main.py
4.2. Преимущества разделения на слои
Аспект	Без слоёв	Со слоями
Замена БД	Переписывать бизнес-логику	Сменить репозиторий
Тестирование	Нужна реальная БД	Моки репозитория
Добавление API	Изменять всё	Добавить адаптер
Понимание кода	Спагетти	Понятная структура
4.3. Когда нужно применять Clean Architecture
Применять:

Крупные проекты (1000+ строк)

Проекты с долгим жизненным циклом

Проекты с частыми изменениями требований

Командная разработка

Не применять:

Маленькие скрипты (1-5 файлов)

Прототипы

Одноразовые задачи

Шпаргалка по SOLID и Clean Architecture
python
# === SOLID ===

# S: Single Responsibility
class User:      # Только данные
class UserSaver: # Только сохранение
class EmailSender: # Только отправка

# O: Open/Closed
class Discount(ABC):
    @abstractmethod
    def apply(self, price: float) -> float: pass

# L: Liskov Substitution
# Подклассы должны заменять родительский класс без ошибок

# I: Interface Segregation
class Workable(ABC): pass
class Eatable(ABC): pass

# D: Dependency Inversion
class Service:
    def __init__(self, repo: Repository):  # Зависимость от абстракции
        self.repo = repo


# === CLEAN ARCHITECTURE ===

# Entities - ядро бизнес-логики
class Order:
    def calculate_total(self): pass

# Use Cases - сценарии использования
class OrderService:
    def __init__(self, repo: OrderRepository):
        self.repo = repo

# Interface Adapters - адаптеры
class SQLiteOrderRepository(OrderRepository): pass

# Frameworks - внешние зависимости
# Flask, FastAPI, Django
Контрольные вопросы
Что такое SRP? Приведите пример нарушения и исправления.

Как принцип Open/Closed помогает при добавлении новых фич?

Почему нарушение LSP может быть незаметно на ранних этапах?

В чём разница между ISP и просто маленькими интерфейсами?

Как Dependency Injection связан с DIP?

Какие слои выделяются в Clean Architecture и как они зависят друг от друга?

Почему Entities не должны зависеть от внешних библиотек?

Как тестирование Unit-тестами связано с принципом DIP?

Практическое задание
Задание: Возьмите любой "плохой" класс из предыдущих работ (например, монолит из ПЗ 2.4) и примените принципы SOLID:

Выделите классы с единственной ответственностью (SRP)

Создайте абстракции для расширения (OCP)

Убедитесь, что подклассы заменяют родительские (LSP)

Разделите большие интерфейсы (ISP)

Внедрите зависимости через абстракции (DIP)

Итог лекции
Вы сегодня:

Изучили пять принципов SOLID с примерами на Python

Поняли, как строить чиcтую архитектуру (Clean Architecture)

Узнали о правильном разделении кода на слои

Научились проектировать тестируемый и расширяемый код

Теперь вы можете создавать профессиональные проекты, которые легко поддерживать и расширять!

