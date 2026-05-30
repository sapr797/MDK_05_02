# ПЗ 2.7. Анализ кода на соответствие SOLID. Рефакторинг с применением SOLID

**Тема:** Принципы SOLID, чистая архитектура, рефакторинг

**Цель работы:**  
Научиться анализировать код на соответствие принципам SOLID, выявлять нарушения и исправлять их, применяя правильные архитектурные паттерны.

**Время выполнения:** 90-120 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `pylint`, `radon`, `mypy`

```bash
pip install pylint radon mypy
Нулевой вариант (эталонный) — Демонстрация преподавателя
Назначение: преподаватель выполняет этот вариант на занятии, показывая полный цикл анализа и исправления кода с применением принципов SOLID.

Исходный "плохой" код (нарушает все принципы SOLID)
python
# bad_solid_demo.py
"""
Плохой код, нарушающий все принципы SOLID.
"""

import json
import sqlite3
import smtplib
from datetime import datetime


class UserManager:
    """
    Огромный класс, который делает ВСЁ:
    - хранит данные пользователя
    - работает с БД
    - отправляет email
    - генерирует отчёты
    - валидирует данные
    - логирует действия
    """
    
    def __init__(self, db_path: str = "users.db", smtp_config: dict = None):
        self.db_path = db_path
        self.smtp_config = smtp_config or {"host": "smtp.gmail.com", "port": 587}
        self.name = None
        self.email = None
        self.role = None
        self._init_db()
    
    def _init_db(self):
        """Инициализация БД"""
        self.conn = sqlite3.connect(self.db_path)
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY,
                name TEXT,
                email TEXT UNIQUE,
                role TEXT
            )
        """)
    
    def set_user_data(self, name: str, email: str, role: str = "user"):
        """Установка данных пользователя"""
        if '@' not in email:
            raise ValueError("Invalid email")
        self.name = name
        self.email = email
        self.role = role
    
    def save_user(self):
        """Сохранение в БД"""
        if not self.name or not self.email:
            raise ValueError("No user data")
        self.conn.execute(
            "INSERT INTO users (name, email, role) VALUES (?, ?, ?)",
            (self.name, self.email, self.role)
        )
        self.conn.commit()
        self._send_welcome_email()
        self._log_action("user_saved")
    
    def _send_welcome_email(self):
        """Отправка приветственного email"""
        server = smtplib.SMTP(self.smtp_config["host"], self.smtp_config["port"])
        server.starttls()
        server.sendmail(
            "noreply@example.com",
            self.email,
            f"Subject: Welcome {self.name}\n\nWelcome to our service!"
        )
        server.quit()
    
    def _log_action(self, action: str):
        """Логирование действия"""
        with open("log.txt", "a") as f:
            f.write(f"{datetime.now()}: {action} for {self.email}\n")
    
    def get_user_report(self) -> str:
        """Генерация отчёта"""
        return f"User: {self.name}, Email: {self.email}, Role: {self.role}"
    
    def delete_user(self, email: str):
        """Удаление пользователя (только для админа)"""
        if self.role != "admin":
            raise PermissionError("Only admin can delete users")
        self.conn.execute("DELETE FROM users WHERE email = ?", (email,))
        self.conn.commit()
        self._log_action(f"user_deleted: {email}")
    
    def get_all_users(self):
        """Получение всех пользователей"""
        cursor = self.conn.execute("SELECT name, email, role FROM users")
        return cursor.fetchall()


# Использование
if __name__ == "__main__":
    manager = UserManager()
    manager.set_user_data("Alice", "alice@example.com", "admin")
    manager.save_user()
    
    manager.set_user_data("Bob", "bob@example.com", "user")
    manager.save_user()
    
    print(manager.get_all_users())
    
    manager.delete_user("bob@example.com")  # Это работает, но почему? role у manager не admin!
Анализ нарушений SOLID
Принцип	Нарушение	Описание
S (SRP)	6+ ответственностей	Класс отвечает за: хранение данных, БД, email, логирование, отчёты, валидацию
O (OCP)	Открыт для модификации	Чтобы добавить новое хранилище (PostgreSQL), нужно менять класс
L (LSP)	Нарушен косвенно	Нет иерархии, нельзя заменить компоненты
I (ISP)	Огромный интерфейс	Класс имеет методы, не нужные в некоторых контекстах
D (DIP)	Зависит от конкретных реализаций	Прямые зависимости от sqlite3, smtplib, файловой системы
Исправленный код (соблюдает SOLID)
python
#!/usr/bin/env python3
"""
Модуль с правильной архитектурой, соблюдающей принципы SOLID.
"""

from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import List, Optional
from datetime import datetime
import sqlite3
import smtplib


# ============================================================
# СЛОЙ 1: ENTITIES (Модели) — SRP соблюдён
# ============================================================

@dataclass
class User:
    """Бизнес-сущность пользователя (только данные)"""
    id: Optional[int]
    name: str
    email: str
    role: str
    
    @staticmethod
    def is_valid_email(email: str) -> bool:
        """Проверка формата email"""
        return '@' in email
    
    def change_email(self, new_email: str) -> None:
        """Бизнес-правило: email должен быть валидным"""
        if not self.is_valid_email(new_email):
            raise ValueError("Invalid email format")
        self.email = new_email
    
    def is_admin(self) -> bool:
        """Проверка прав администратора"""
        return self.role == "admin"
    
    def can_delete(self, target: 'User') -> bool:
        """Админ может удалить любого, кроме себя"""
        if not self.is_admin():
            return False
        return self.id != target.id


# ============================================================
# СЛОЙ 2: INTERFACES (Абстракции) — DIP соблюдён
# ============================================================

class UserRepository(ABC):
    """Абстракция для работы с хранилищем пользователей"""
    
    @abstractmethod
    def save(self, user: User) -> User:
        pass
    
    @abstractmethod
    def find_by_email(self, email: str) -> Optional[User]:
        pass
    
    @abstractmethod
    def find_by_id(self, user_id: int) -> Optional[User]:
        pass
    
    @abstractmethod
    def delete(self, email: str) -> bool:
        pass
    
    @abstractmethod
    def get_all(self) -> List[User]:
        pass


class EmailSender(ABC):
    """Абстракция для отправки email"""
    
    @abstractmethod
    def send_welcome(self, user: User) -> None:
        pass
    
    @abstractmethod
    def send_notification(self, user: User, message: str) -> None:
        pass


class Logger(ABC):
    """Абстракция для логирования"""
    
    @abstractmethod
    def log(self, level: str, message: str) -> None:
        pass


class ReportGenerator(ABC):
    """Абстракция для генерации отчётов"""
    
    @abstractmethod
    def generate(self, user: User) -> str:
        pass


# ============================================================
# СЛОЙ 3: CONCRETE IMPLEMENTATIONS (Реализации) — OCP соблюдён
# ============================================================

class SQLiteUserRepository(UserRepository):
    """Реализация репозитория для SQLite"""
    
    def __init__(self, db_path: str = "users.db"):
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
                (user.name, user.email, user.role)
            )
            user.id = cursor.lastrowid
        return user
    
    def find_by_email(self, email: str) -> Optional[User]:
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                "SELECT id, name, email, role FROM users WHERE email = ?",
                (email,)
            )
            row = cursor.fetchone()
            if row:
                return User(id=row[0], name=row[1], email=row[2], role=row[3])
        return None
    
    def find_by_id(self, user_id: int) -> Optional[User]:
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                "SELECT id, name, email, role FROM users WHERE id = ?",
                (user_id,)
            )
            row = cursor.fetchone()
            if row:
                return User(id=row[0], name=row[1], email=row[2], role=row[3])
        return None
    
    def delete(self, email: str) -> bool:
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute("DELETE FROM users WHERE email = ?", (email,))
            return cursor.rowcount > 0
    
    def get_all(self) -> List[User]:
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute("SELECT id, name, email, role FROM users")
            return [
                User(id=row[0], name=row[1], email=row[2], role=row[3])
                for row in cursor.fetchall()
            ]


class SMTPEmailSender(EmailSender):
    """Реализация отправки email через SMTP"""
    
    def __init__(self, host: str = "smtp.gmail.com", port: int = 587):
        self.host = host
        self.port = port
    
    def send_welcome(self, user: User) -> None:
        self._send(user, f"Welcome {user.name}!", "Welcome to our service!")
    
    def send_notification(self, user: User, message: str) -> None:
        self._send(user, "Notification", message)
    
    def _send(self, user: User, subject: str, body: str) -> None:
        # В реальном коде здесь была бы отправка
        print(f"Email sent to {user.email}: {subject}")


class FileLogger(Logger):
    """Реализация логирования в файл"""
    
    def __init__(self, log_path: str = "log.txt"):
        self.log_path = log_path
    
    def log(self, level: str, message: str) -> None:
        timestamp = datetime.now().isoformat()
        with open(self.log_path, "a", encoding='utf-8') as f:
            f.write(f"[{timestamp}] {level}: {message}\n")


class SimpleReportGenerator(ReportGenerator):
    """Простая реализация генерации отчётов"""
    
    def generate(self, user: User) -> str:
        return f"User Report: {user.name} ({user.email}) - Role: {user.role}"


# ============================================================
# СЛОЙ 4: APPLICATION SERVICE (Use Case) — единая ответственность
# ============================================================

class UserService:
    """
    Сервис приложения. Содержит бизнес-логику.
    Зависит от абстракций (DIP), а не от конкретных реализаций.
    """
    
    def __init__(
        self,
        user_repository: UserRepository,
        email_sender: EmailSender,
        logger: Logger,
        report_generator: ReportGenerator
    ):
        self.user_repo = user_repository
        self.email_sender = email_sender
        self.logger = logger
        self.report_generator = report_generator
    
    def register_user(self, name: str, email: str, role: str = "user") -> User:
        """Регистрация нового пользователя"""
        # Проверка, не существует ли уже
        if self.user_repo.find_by_email(email):
            self.logger.log("WARNING", f"Registration attempt for existing email: {email}")
            raise ValueError(f"User with email {email} already exists")
        
        # Создание пользователя
        user = User(id=None, name=name, email=email, role=role)
        
        # Сохранение
        saved_user = self.user_repo.save(user)
        
        # Отправка приветствия
        self.email_sender.send_welcome(saved_user)
        
        # Логирование
        self.logger.log("INFO", f"User registered: {email}")
        
        return saved_user
    
    def get_user_profile(self, email: str) -> Optional[User]:
        """Получение профиля пользователя"""
        user = self.user_repo.find_by_email(email)
        if user:
            self.logger.log("INFO", f"Profile accessed for: {email}")
        return user
    
    def delete_user(self, admin_user: User, target_email: str) -> bool:
        """Удаление пользователя (только для администратора)"""
        target_user = self.user_repo.find_by_email(target_email)
        if not target_user:
            self.logger.log("WARNING", f"Delete attempt for non-existent user: {target_email}")
            return False
        
        if not admin_user.can_delete(target_user):
            self.logger.log("ERROR", f"Unauthorized delete attempt by {admin_user.email}")
            raise PermissionError("Only admin can delete other users")
        
        result = self.user_repo.delete(target_email)
        if result:
            self.logger.log("INFO", f"User deleted: {target_email} by {admin_user.email}")
        return result
    
    def get_all_users(self) -> List[User]:
        """Получение всех пользователей"""
        users = self.user_repo.get_all()
        self.logger.log("INFO", f"Listed all users: {len(users)} records")
        return users
    
    def generate_user_report(self, user: User) -> str:
        """Генерация отчёта о пользователе"""
        return self.report_generator.generate(user)


# ============================================================
# DEPENDENCY INJECTION (Сборка зависимостей)
# ============================================================

def create_user_service(
    db_path: str = "users.db",
    smtp_host: str = "smtp.gmail.com",
    smtp_port: int = 587,
    log_path: str = "log.txt"
) -> UserService:
    """Фабрика для создания сервиса со всеми зависимостями"""
    repository = SQLiteUserRepository(db_path)
    email_sender = SMTPEmailSender(smtp_host, smtp_port)
    logger = FileLogger(log_path)
    report_generator = SimpleReportGenerator()
    
    return UserService(repository, email_sender, logger, report_generator)


# ============================================================
# ПРИМЕР ИСПОЛЬЗОВАНИЯ
# ============================================================

def main():
    """Демонстрация работы"""
    # Создаём сервис через фабрику
    user_service = create_user_service()
    
    print("=" * 60)
    print("ДЕМОНСТРАЦИЯ РАБОТЫ (SOLID версия)")
    print("=" * 60)
    
    # Регистрация пользователей
    print("\n1. Регистрация пользователей:")
    alice = user_service.register_user("Alice", "alice@example.com", "admin")
    bob = user_service.register_user("Bob", "bob@example.com", "user")
    print(f"   Создан: {alice}")
    print(f"   Создан: {bob}")
    
    # Получение профиля
    print("\n2. Получение профиля:")
    profile = user_service.get_user_profile("alice@example.com")
    print(f"   Профиль: {profile}")
    
    # Генерация отчёта
    print("\n3. Генерация отчёта:")
    report = user_service.generate_user_report(alice)
    print(f"   {report}")
    
    # Удаление пользователя (админ удаляет обычного)
    print("\n4. Удаление пользователя (админ -> обычный):")
    try:
        user_service.delete_user(alice, "bob@example.com")
        print(f"   Пользователь bob удалён")
    except PermissionError as e:
        print(f"   Ошибка: {e}")
    
    # Попытка удаления админа обычным пользователем
    print("\n5. Попытка удаления (обычный -> админ):")
    try:
        # Получаем обычного пользователя (не админа)
        # Создадим отдельный сервис или используем существующий
        regular_user = User(id=999, name="Test", email="test@example.com", role="user")
        user_service.delete_user(regular_user, "alice@example.com")
    except PermissionError as e:
        print(f"   Ошибка (ожидаемо): {e}")
    
    print("\n" + "=" * 60)
    print("Демонстрация завершена")
    print("=" * 60)


if __name__ == "__main__":
    main()
Сравнение "до" и "после"
Аспект	До рефакторинга	После рефакторинга
Количество классов	1	8
Количество строк	~80	~300
SRP	❌ 6+ ответственностей	✅ каждый класс имеет одну
OCP	❌ требуется модификация	✅ добавление через реализацию интерфейсов
LSP	❌ нет иерархии	✅ чёткие интерфейсы
ISP	❌ огромный класс	✅ маленькие интерфейсы
DIP	❌ зависимости от конкретных библиотек	✅ зависимости от абстракций
Тестируемость	❌ сложно (нужна БД, SMTP)	✅ легко (моки)
25 вариантов практической работы (ПЗ 2.7)
Общая структура каждого варианта:
Студент получает "плохой" код, нарушающий принципы SOLID, должен проанализировать нарушения и выполнить рефакторинг.

Уровни сложности:

Варианты 1-8: базовый (2-3 нарушения SOLID)

Варианты 9-17: средний (4-5 нарушений)

Варианты 18-25: сложный (все принципы + архитектурные паттерны)

Варианты 1-8 (Базовый уровень)
Вариант 1. Калькулятор скидок
python
# variant_1.py
class DiscountCalculator:
    def __init__(self, customer_type):
        self.customer_type = customer_type
    
    def calculate(self, price):
        if self.customer_type == "regular":
            return price * 0.95
        elif self.customer_type == "vip":
            return price * 0.9
        elif self.customer_type == "employee":
            return price * 0.8
        return price
Нарушения: OCP (нужно менять код для добавления скидок)

Задание: Применить OCP через стратегию.

Вариант 2. Отправка уведомлений
python
# variant_2.py
class NotificationService:
    def send(self, message, channel):
        if channel == "email":
            print(f"Sending email: {message}")
        elif channel == "sms":
            print(f"Sending SMS: {message}")
        elif channel == "push":
            print(f"Sending push: {message}")
Нарушения: OCP, SRP

Задание: Создать иерархию каналов уведомлений.

Вариант 3. Обработчик заказов
python
# variant_3.py
class OrderProcessor:
    def process(self, order):
        self.validate(order)
        self.calculate_tax(order)
        self.save_to_db(order)
        self.send_email(order)
        self.print_receipt(order)
    
    def validate(self, order): pass
    def calculate_tax(self, order): pass
    def save_to_db(self, order): pass
    def send_email(self, order): pass
    def print_receipt(self, order): pass
Нарушения: SRP (слишком много ответственностей)

Задание: Разделить на отдельные классы.

Вариант 4. Менеджер пользователей (упрощённый)
python
# variant_4.py
class UserManager:
    def __init__(self):
        self.users = []
    
    def add_user(self, name, email):
        if '@' not in email:
            raise ValueError("Invalid email")
        self.users.append({"name": name, "email": email})
        self._save_to_file()
    
    def _save_to_file(self):
        with open("users.txt", "w") as f:
            for u in self.users:
                f.write(f"{u['name']},{u['email']}\n")
Нарушения: SRP (данные + валидация + сохранение)

Задание: Выделить валидатор и репозиторий.

Вариант 5. Логгер с разными выходами
python
# variant_5.py
class Logger:
    def log(self, message, output):
        if output == "console":
            print(message)
        elif output == "file":
            with open("log.txt", "a") as f:
                f.write(message + "\n")
        elif output == "database":
            # код сохранения в БД
            pass
Нарушения: OCP, SRP

Задание: Применить паттерн Стратегия.

Вариант 6. Парсер документов
python
# variant_6.py
class DocumentParser:
    def parse(self, filename, filetype):
        if filetype == "pdf":
            # парсинг PDF
            return "PDF content"
        elif filetype == "docx":
            # парсинг DOCX
            return "DOCX content"
        elif filetype == "txt":
            # парсинг TXT
            return "TXT content"
Нарушения: OCP

Задание: Создать абстрактный класс Parser и реализации.

Вариант 7. Валидатор форм
python
# variant_7.py
class FormValidator:
    def validate(self, data):
        errors = []
        if not data.get('name'):
            errors.append("Name required")
        if not data.get('email'):
            errors.append("Email required")
        elif '@' not in data['email']:
            errors.append("Invalid email")
        if len(data.get('phone', '')) < 10:
            errors.append("Phone too short")
        return errors
Нарушения: SRP, OCP

Задание: Создать отдельные валидаторы для каждого поля.

Вариант 8. Репозиторий (жёсткая привязка)
python
# variant_8.py
import sqlite3

class UserRepository:
    def __init__(self):
        self.conn = sqlite3.connect("users.db")
    
    def save(self, user):
        self.conn.execute("INSERT INTO users VALUES (?, ?)", (user.name, user.email))
Нарушения: DIP (зависимость от конкретной БД)

Задание: Внедрить абстракцию Database.

Варианты 9-17 (Средний уровень)
Вариант 9. Система плагинов
python
# variant_9.py
class PluginSystem:
    def __init__(self):
        self.plugins = []
    
    def load_plugin(self, name):
        if name == "plugin_a":
            self.plugins.append(PluginA())
        elif name == "plugin_b":
            self.plugins.append(PluginB())
    
    def execute_all(self):
        for p in self.plugins:
            p.run()
Нарушения: OCP, SRP

Задание: Реализовать через абстрактный класс Plugin и динамическую загрузку.

Вариант 10. Кэш с разными стратегиями
python
# variant_10.py
class Cache:
    def __init__(self, strategy="lru"):
        self.strategy = strategy
        self.data = {}
    
    def set(self, key, value):
        if self.strategy == "lru":
            # LRU логика
            pass
        elif self.strategy == "fifo":
            # FIFO логика
            pass
Нарушения: OCP

Задание: Применить паттерн Стратегия.

Вариант 11. Экспортёр данных
python
# variant_11.py
class DataExporter:
    def export(self, data, format, destination):
        if format == "json":
            json_data = json.dumps(data)
            if destination == "file":
                with open("export.json", "w") as f:
                    f.write(json_data)
            elif destination == "api":
                requests.post("https://api.example.com", json=data)
        elif format == "csv":
            # аналогично
            pass
Нарушения: SRP, OCP (2 нарушения)

Задание: Разделить на форматтеры и дестинации.

Вариант 12. Аутентификация
python
# variant_12.py
class AuthService:
    def __init__(self):
        self.users = {}
    
    def register(self, username, password):
        if len(password) < 6:
            raise ValueError("Password too short")
        self.users[username] = password
        self._save_to_db()
    
    def login(self, username, password):
        return self.users.get(username) == password
Нарушения: SRP (валидация + хранение + логика)

Задание: Разделить на Validator, Repository, AuthService.

Вариант 13. Обработка платежей
python
# variant_13.py
class PaymentProcessor:
    def process(self, amount, method, card_data=None):
        if method == "card":
            # обработка карты
            self._charge_card(amount, card_data)
            self._send_receipt(card_data['email'])
        elif method == "paypal":
            # обработка PayPal
            self._charge_paypal(amount)
            self._send_receipt()
        self._log_transaction(amount, method)
Нарушения: SRP, OCP, DIP

Задание: Создать иерархию платежных методов.

Вариант 14. Система кэширования с хранением
python
# variant_14.py
class CacheSystem:
    def __init__(self, storage="memory"):
        if storage == "memory":
            self.cache = {}
        elif storage == "redis":
            import redis
            self.cache = redis.Redis()
    
    def get(self, key):
        if isinstance(self.cache, dict):
            return self.cache.get(key)
        else:
            return self.cache.get(key)
Нарушения: DIP, OCP

Задание: Создать абстракцию Storage.

Вариант 15. Логирование с метриками
python
# variant_15.py
class LoggerWithMetrics:
    def log(self, level, message):
        # Логирование в файл
        with open("app.log", "a") as f:
            f.write(f"{level}: {message}\n")
        
        # Отправка метрик
        if level == "ERROR":
            self._send_metric(message)
    
    def _send_metric(self, message):
        requests.post("https://metrics.example.com", json={"error": message})
Нарушения: SRP, DIP

Задание: Разделить на Logger и MetricsSender.

Вариант 16. Генератор отчётов
python
# variant_16.py
class ReportGenerator:
    def generate(self, data, report_type):
        if report_type == "pdf":
            from reportlab import ...
            # генерация PDF
        elif report_type == "html":
            # генерация HTML
        elif report_type == "excel":
            # генерация Excel
Нарушения: OCP

Задание: Создать абстрактный класс Report и реализации.

Вариант 17. Сервис уведомлений с шаблонами
python
# variant_17.py
class NotificationService:
    def send(self, user, template_name, context):
        if template_name == "welcome":
            message = f"Welcome {user.name}!"
        elif template_name == "reset_password":
            message = f"Reset your password, {user.name}!"
        elif template_name == "invoice":
            message = f"Invoice #{context['invoice_id']} for {user.name}"
        
        # Отправка через email
        self._send_email(user.email, message)
Нарушения: SRP, OCP

Задание: Выделить TemplateEngine и EmailSender.

Варианты 18-25 (Сложный уровень)
Вариант 18. ETL-процесс
python
# variant_18.py
class ETLProcessor:
    def run(self):
        # Extract
        data = self._read_csv("data.csv")
        
        # Transform
        transformed = []
        for row in data:
            if row['value'] > 0:
                row['processed'] = row['value'] * 2
                transformed.append(row)
        
        # Load
        with open("output.json", "w") as f:
            json.dump(transformed, f)
    
    def _read_csv(self, path):
        import csv
        with open(path) as f:
            return list(csv.DictReader(f))
Нарушения: SRP, OCP, DIP

Задание: Реализовать паттерн Pipeline с отдельными E, T, L компонентами.

Вариант 19. Веб-сервер
python
# variant_19.py
class WebServer:
    def __init__(self, port=8080):
        self.port = port
        self.routes = {}
    
    def route(self, path, method, handler):
        self.routes[f"{method}:{path}"] = handler
    
    def run(self):
        import socket
        sock = socket.socket()
        sock.bind(('localhost', self.port))
        sock.listen(5)
        
        while True:
            conn, addr = sock.accept()
            request = conn.recv(1024).decode()
            # парсинг HTTP запроса
            method, path, _ = request.split(' ')[:3]
            handler = self.routes.get(f"{method}:{path}")
            if handler:
                response = handler()
            else:
                response = "404 Not Found"
            conn.send(f"HTTP/1.1 200 OK\n\n{response}".encode())
            conn.close()
Нарушения: SRP, OCP, DIP

Задание: Разделить на сокет-сервер, маршрутизатор, обработчики.

Вариант 20. Менеджер задач
python
# variant_20.py
class TaskManager:
    def __init__(self):
        self.tasks = []
        self.observers = []
    
    def add_task(self, title, priority):
        task = {"title": title, "priority": priority, "status": "pending"}
        self.tasks.append(task)
        
        # Уведомление
        for obs in self.observers:
            obs.update(task)
        
        # Сохранение
        self._save()
        
        # Логирование
        print(f"Task added: {title}")
    
    def complete_task(self, index):
        self.tasks[index]["status"] = "done"
        self._save()
        print(f"Task completed: {self.tasks[index]['title']}")
Нарушения: SRP, OCP, DIP

Задание: Применить паттерны Наблюдатель, Репозиторий, Сервис.

Вариант 21. Анализатор логов
python
# variant_21.py
class LogAnalyzer:
    def analyze(self, log_file):
        errors = 0
        warnings = 0
        info = 0
        
        with open(log_file) as f:
            for line in f:
                if "ERROR" in line:
                    errors += 1
                elif "WARNING" in line:
                    warnings += 1
                elif "INFO" in line:
                    info += 1
        
        report = {
            "errors": errors,
            "warnings": warnings,
            "info": info,
            "total": errors + warnings + info
        }
        
        with open("report.json", "w") as f:
            json.dump(report, f)
        
        return report
Нарушения: SRP, OCP

Задание: Создать парсер логов, анализатор, экспортёр отчёта.

Вариант 22. API клиент
python
# variant_22.py
class APIClient:
    def __init__(self, base_url):
        self.base_url = base_url
        self.cache = {}
    
    def get(self, endpoint):
        if endpoint in self.cache:
            return self.cache[endpoint]
        
        response = requests.get(f"{self.base_url}/{endpoint}")
        if response.status_code == 200:
            self.cache[endpoint] = response.json()
            return response.json()
        return None
    
    def post(self, endpoint, data):
        response = requests.post(f"{self.base_url}/{endpoint}", json=data)
        return response.json() if response.status_code == 201 else None
Нарушения: SRP, OCP

Задание: Выделить HTTPClient, Cache, Serializer.

Вариант 23. Конфигуратор приложения
python
# variant_23.py
class ConfigManager:
    def __init__(self):
        self.config = {}
        self._load()
    
    def _load(self):
        # Загрузка из файла
        if os.path.exists("config.json"):
            with open("config.json") as f:
                self.config = json.load(f)
        
        # Загрузка из переменных окружения
        self.config['debug'] = os.getenv('DEBUG', 'False').lower() == 'true'
        self.config['port'] = int(os.getenv('PORT', '8080'))
    
    def get(self, key, default=None):
        return self.config.get(key, default)
    
    def set(self, key, value):
        self.config[key] = value
        self._save()
    
    def _save(self):
        with open("config.json", "w") as f:
            json.dump(self.config, f)
Нарушения: SRP (файл + env), DIP

Задание: Создать источники конфигурации (FileSource, EnvSource).

Вариант 24. Система аутентификации с MFA
python
# variant_24.py
class AuthSystem:
    def __init__(self):
        self.users = {}
        self.tokens = {}
    
    def register(self, username, password, phone):
        if len(password) < 8:
            raise ValueError("Weak password")
        self.users[username] = {"password": password, "phone": phone}
        self._send_sms(phone, "Welcome!")
    
    def login(self, username, password):
        if self.users.get(username, {}).get("password") != password:
            return False
        # MFA
        code = random.randint(100000, 999999)
        self._send_sms(self.users[username]["phone"], f"Code: {code}")
        user_code = input("Enter code: ")
        return user_code == str(code)
Нарушения: SRP, DIP, OCP

Задание: Разделить на UserRepository, PasswordValidator, MFAService.

Вариант 25. Планировщик задач (самый сложный)
python
# variant_25.py
class Scheduler:
    def __init__(self):
        self.tasks = []
    
    def add_task(self, task_func, interval_seconds, params=None):
        self.tasks.append({
            "func": task_func,
            "interval": interval_seconds,
            "params": params or {},
            "last_run": None
        })
    
    def run_forever(self):
        import time
        while True:
            now = time.time()
            for task in self.tasks:
                last = task["last_run"] or 0
                if now - last >= task["interval"]:
                    try:
                        task["func"](**task["params"])
                        task["last_run"] = now
                    except Exception as e:
                        print(f"Task failed: {e}")
                        self._log_error(e)
            time.sleep(1)
    
    def _log_error(self, error):
        with open("scheduler_errors.log", "a") as f:
            f.write(f"{datetime.now()}: {error}\n")
Нарушения: SRP, OCP, DIP (все принципы)

Задание: Создать отдельные компоненты: Task, Scheduler, Executor, Logger.

Карточка студента (шаблон)
text
ПЗ 2.7. АНАЛИЗ КОДА НА СООТВЕТСТВИЕ SOLID

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== ВЫЯВЛЕННЫЕ НАРУШЕНИЯ ===

S (SRP): _________________________________
O (OCP): _________________________________
L (LSP): _________________________________
I (ISP): _________________________________
D (DIP): _________________________________

=== ПЛАН РЕФАКТОРИНГА ===

Новые классы/модули:
1. _______________________________
2. _______________________________
3. _______________________________

Используемые паттерны:
□ Стратегия □ Репозиторий □ Внедрение зависимостей
□ Фабрика □ Наблюдатель □ Другое: _________

=== РЕЗУЛЬТАТ ===

Количество классов до: ___ после: ___
Pylint оценка до: ___/10 после: ___/10

Дата выполнения: _____________
Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	SOLID не применён, код не работает
3 (удовлетворительно)	Исправлены 2-3 нарушения, код работает
4 (хорошо)	Исправлены все нарушения, код соответствует SOLID
5 (отлично)	Полное соответствие SOLID, добавлены абстракции, DI, тесты
Шпаргалка по SOLID для рефакторинга
python
# === ПРИЗНАКИ НАРУШЕНИЙ ===

# S: Класс с методами, которые не связаны логически
class GodClass:
    def save_to_db(self): pass
    def send_email(self): pass
    def generate_report(self): pass

# O: Цепочка if/elif по типу
if type == "A":
    do_a()
elif type == "B":
    do_b()

# L: Метод, который выбрасывает NotImplementedError
class Bird:
    def fly(self): raise NotImplementedError

# I: Класс реализует методы, которые не использует
class Worker(ABC):
    @abstractmethod
    def work(self): pass
    @abstractmethod
    def eat(self): pass
    @abstractmethod
    def sleep(self): pass

class Robot(Worker):
    def work(self): pass
    def eat(self): raise NotImplementedError  # нарушение ISP

# D: Прямое создание зависимостей внутри класса
class Service:
    def __init__(self):
        self.db = MySQLDatabase()  # нарушение DIP
