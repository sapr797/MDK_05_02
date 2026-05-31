# ПЗ 2.28. Создание моделей данных для API (User, Order, Product)

**Тема:** Работа с базами данных, SQLAlchemy ORM, модели, связи между таблицами

**Цель работы:**  
Научиться создавать модели данных для API с использованием SQLAlchemy, настраивать связи между таблицами (User, Order, Product), реализовывать CRUD операции.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `sqlalchemy`, `alembic`, `passlib`, `pydantic`

```bash
pip install sqlalchemy alembic passlib pydantic
Главная мысль: Хорошо спроектированные модели данных — это фундамент надёжного API. Продуманные связи между сущностями упрощают разработку и поддержку.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Связи между моделями
Тип связи	Описание	Пример
Один-к-одному	Одна запись связана с одной записью	User → Profile
Один-ко-многим	Одна запись связана с несколькими	User → Orders
Многие-ко-многим	Многие записи связаны со многими	Order → Product
1.2. Схема базы данных для API
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                              СХЕМА БАЗЫ ДАННЫХ                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────┐          ┌─────────┐          ┌─────────────┐                 │
│  │  User   │          │  Order  │          │   Product   │                 │
│  ├─────────┤          ├─────────┤          ├─────────────┤                 │
│  │ id      │◄─────────│ user_id │          │ id          │                 │
│  │ name    │          │ id      │───────┐  │ name        │                 │
│  │ email   │          │ status  │       │  │ price       │                 │
│  │ password│          │ total   │       │  │ stock       │                 │
│  └─────────┘          └─────────┘       │  └─────────────┘                 │
│        │                    │          │         │                         │
│        │                    │          │         │                         │
│        ▼                    ▼          │         ▼                         │
│  ┌─────────┐          ┌─────────────┐  │  ┌─────────────┐                 │
│  │ Profile │          │ OrderItem   │  │  │  Category   │                 │
│  ├─────────┤          ├─────────────┤  │  ├─────────────┤                 │
│  │ user_id │◄─────────│ order_id    │  │  │ id          │                 │
│  │ bio     │          │ product_id  │──┘  │ name        │                 │
│  │ avatar  │          │ quantity    │     └─────────────┘                 │
│  └─────────┘          │ price       │            │                         │
│                       └─────────────┘            │                         │
│                              │                   │                         │
│                              │                   │                         │
│                              ▼                   ▼                         │
│                       Связь многие-ко-многим  Связь многие-ко-одному       │
│                       через OrderItem         Product → Category           │
└─────────────────────────────────────────────────────────────────────────────┘
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая создание моделей данных для API.

Техническое задание (нулевой вариант)
Создайте модели данных для интернет-магазина с использованием SQLAlchemy:

User (id, name, email, password_hash, is_active, created_at)

Profile (id, user_id, bio, avatar_url, phone) — связь один-к-одному с User

Category (id, name, slug, description)

Product (id, name, description, price, stock, category_id) — связь с Category

Order (id, user_id, status, total, created_at) — связь с User

OrderItem (id, order_id, product_id, quantity, price) — связь многие-ко-многим

Эталонная реализация
python
# models.py
"""
Модели данных для интернет-магазина (SQLAlchemy ORM).
"""

from sqlalchemy import (
    create_engine, Column, Integer, String, Float, Boolean, 
    DateTime, Text, ForeignKey, Numeric, Table
)
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship, sessionmaker
from datetime import datetime
from passlib.context import CryptContext
from typing import List, Optional

# ============================================================
# НАСТРОЙКА БАЗЫ ДАННЫХ
# ============================================================

# URL подключения к БД
DATABASE_URL = "sqlite:///./shop.db"

# Создание движка
engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False},  # для SQLite
    echo=True  # логирование SQL запросов
)

# Базовый класс для моделей
Base = declarative_base()

# Фабрика сессий
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Хэширование паролей
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


# ============================================================
# ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ
# ============================================================

def get_db():
    """Генератор сессий для dependency injection."""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


def hash_password(password: str) -> str:
    """Хэширует пароль."""
    return pwd_context.hash(password)


def verify_password(plain_password: str, hashed_password: str) -> bool:
    """Проверяет пароль."""
    return pwd_context.verify(plain_password, hashed_password)


# ============================================================
# МОДЕЛЬ USER (Пользователь)
# ============================================================

class User(Base):
    """Модель пользователя."""
    
    __tablename__ = "users"
    
    # Поля
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True, nullable=False, index=True)
    password_hash = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    is_admin = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
    
    # Связи
    profile = relationship("Profile", back_populates="user", uselist=False, cascade="all, delete-orphan")
    orders = relationship("Order", back_populates="user", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<User(id={self.id}, name={self.name}, email={self.email})>"
    
    def set_password(self, password: str):
        """Устанавливает хэш пароля."""
        self.password_hash = hash_password(password)
    
    def check_password(self, password: str) -> bool:
        """Проверяет пароль."""
        return verify_password(password, self.password_hash)
    
    def to_dict(self) -> dict:
        """Преобразует в словарь (без пароля)."""
        return {
            "id": self.id,
            "name": self.name,
            "email": self.email,
            "is_active": self.is_active,
            "is_admin": self.is_admin,
            "created_at": self.created_at.isoformat() if self.created_at else None
        }


# ============================================================
# МОДЕЛЬ PROFILE (Профиль пользователя)
# ============================================================

class Profile(Base):
    """Модель профиля пользователя (связь один-к-одному)."""
    
    __tablename__ = "profiles"
    
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"), unique=True, nullable=False)
    bio = Column(Text, nullable=True)
    avatar_url = Column(String(500), nullable=True)
    phone = Column(String(20), nullable=True)
    address = Column(String(500), nullable=True)
    city = Column(String(100), nullable=True)
    country = Column(String(100), default="Россия")
    
    # Связь с пользователем
    user = relationship("User", back_populates="profile")
    
    def __repr__(self):
        return f"<Profile(user_id={self.user_id}, phone={self.phone})>"
    
    def to_dict(self) -> dict:
        """Преобразует в словарь."""
        return {
            "bio": self.bio,
            "avatar_url": self.avatar_url,
            "phone": self.phone,
            "address": self.address,
            "city": self.city,
            "country": self.country
        }


# ============================================================
# МОДЕЛЬ CATEGORY (Категория товаров)
# ============================================================

class Category(Base):
    """Модель категории товаров."""
    
    __tablename__ = "categories"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False, unique=True)
    slug = Column(String(100), nullable=False, unique=True, index=True)
    description = Column(Text, nullable=True)
    
    # Связь с товарами
    products = relationship("Product", back_populates="category", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<Category(id={self.id}, name={self.name})>"
    
    def to_dict(self) -> dict:
        """Преобразует в словарь."""
        return {
            "id": self.id,
            "name": self.name,
            "slug": self.slug,
            "description": self.description
        }


# ============================================================
# МОДЕЛЬ PRODUCT (Товар)
# ============================================================

class Product(Base):
    """Модель товара."""
    
    __tablename__ = "products"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(200), nullable=False, index=True)
    description = Column(Text, nullable=True)
    price = Column(Numeric(10, 2), nullable=False)
    stock = Column(Integer, default=0)
    is_active = Column(Boolean, default=True)
    image_url = Column(String(500), nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
    
    # Внешний ключ на категорию
    category_id = Column(Integer, ForeignKey("categories.id"), nullable=True)
    
    # Связи
    category = relationship("Category", back_populates="products")
    order_items = relationship("OrderItem", back_populates="product", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<Product(id={self.id}, name={self.name}, price={self.price})>"
    
    def to_dict(self) -> dict:
        """Преобразует в словарь."""
        return {
            "id": self.id,
            "name": self.name,
            "description": self.description,
            "price": float(self.price),
            "stock": self.stock,
            "is_active": self.is_active,
            "image_url": self.image_url,
            "category_id": self.category_id,
            "category_name": self.category.name if self.category else None,
            "created_at": self.created_at.isoformat() if self.created_at else None
        }
    
    def reduce_stock(self, quantity: int) -> bool:
        """Уменьшает количество товара на складе."""
        if self.stock >= quantity:
            self.stock -= quantity
            return True
        return False
    
    def increase_stock(self, quantity: int):
        """Увеличивает количество товара на складе."""
        self.stock += quantity


# ============================================================
# МОДЕЛЬ ORDER (Заказ)
# ============================================================

class OrderStatus:
    """Статусы заказа."""
    PENDING = "pending"
    PROCESSING = "processing"
    SHIPPED = "shipped"
    DELIVERED = "delivered"
    CANCELLED = "cancelled"
    
    @classmethod
    def choices(cls):
        return [
            cls.PENDING,
            cls.PROCESSING,
            cls.SHIPPED,
            cls.DELIVERED,
            cls.CANCELLED
        ]


class Order(Base):
    """Модель заказа."""
    
    __tablename__ = "orders"
    
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    status = Column(String(20), default=OrderStatus.PENDING)
    total = Column(Numeric(10, 2), default=0.00)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
    
    # Связи
    user = relationship("User", back_populates="orders")
    items = relationship("OrderItem", back_populates="order", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<Order(id={self.id}, user_id={self.user_id}, total={self.total}, status={self.status})>"
    
    def to_dict(self) -> dict:
        """Преобразует в словарь."""
        return {
            "id": self.id,
            "user_id": self.user_id,
            "status": self.status,
            "total": float(self.total),
            "items": [item.to_dict() for item in self.items],
            "created_at": self.created_at.isoformat() if self.created_at else None,
            "updated_at": self.updated_at.isoformat() if self.updated_at else None
        }
    
    def calculate_total(self):
        """Пересчитывает общую сумму заказа."""
        self.total = sum(item.price * item.quantity for item in self.items)
        return self.total
    
    def can_cancel(self) -> bool:
        """Проверяет, можно ли отменить заказ."""
        return self.status in [OrderStatus.PENDING, OrderStatus.PROCESSING]


# ============================================================
# МОДЕЛЬ ORDER_ITEM (Товар в заказе)
# ============================================================

class OrderItem(Base):
    """Модель товара в заказе (связующая таблица)."""
    
    __tablename__ = "order_items"
    
    id = Column(Integer, primary_key=True, index=True)
    order_id = Column(Integer, ForeignKey("orders.id"), nullable=False)
    product_id = Column(Integer, ForeignKey("products.id"), nullable=False)
    quantity = Column(Integer, nullable=False, default=1)
    price = Column(Numeric(10, 2), nullable=False)  # цена на момент покупки
    
    # Связи
    order = relationship("Order", back_populates="items")
    product = relationship("Product", back_populates="order_items")
    
    def __repr__(self):
        return f"<OrderItem(order_id={self.order_id}, product_id={self.product_id}, quantity={self.quantity})>"
    
    def to_dict(self) -> dict:
        """Преобразует в словарь."""
        return {
            "id": self.id,
            "product_id": self.product_id,
            "product_name": self.product.name if self.product else None,
            "quantity": self.quantity,
            "price": float(self.price),
            "subtotal": float(self.price * self.quantity)
        }


# ============================================================
# CRUD ОПЕРАЦИИ (для примера)
# ============================================================

class UserCRUD:
    """CRUD операции для модели User."""
    
    @staticmethod
    def create(db, name: str, email: str, password: str, **kwargs) -> User:
        """Создание пользователя."""
        user = User(
            name=name,
            email=email.lower(),
            **kwargs
        )
        user.set_password(password)
        db.add(user)
        db.commit()
        db.refresh(user)
        return user
    
    @staticmethod
    def get_by_id(db, user_id: int) -> Optional[User]:
        """Получение пользователя по ID."""
        return db.query(User).filter(User.id == user_id).first()
    
    @staticmethod
    def get_by_email(db, email: str) -> Optional[User]:
        """Получение пользователя по email."""
        return db.query(User).filter(User.email == email.lower()).first()
    
    @staticmethod
    def get_all(db, skip: int = 0, limit: int = 100) -> List[User]:
        """Получение всех пользователей."""
        return db.query(User).offset(skip).limit(limit).all()
    
    @staticmethod
    def update(db, user_id: int, **kwargs) -> Optional[User]:
        """Обновление пользователя."""
        user = UserCRUD.get_by_id(db, user_id)
        if user:
            for key, value in kwargs.items():
                if hasattr(user, key) and key != "password_hash":
                    setattr(user, key, value)
            if "password" in kwargs:
                user.set_password(kwargs["password"])
            db.commit()
            db.refresh(user)
        return user
    
    @staticmethod
    def delete(db, user_id: int) -> bool:
        """Удаление пользователя."""
        user = UserCRUD.get_by_id(db, user_id)
        if user:
            db.delete(user)
            db.commit()
            return True
        return False


class ProductCRUD:
    """CRUD операции для модели Product."""
    
    @staticmethod
    def create(db, name: str, price: float, **kwargs) -> Product:
        """Создание товара."""
        product = Product(name=name, price=price, **kwargs)
        db.add(product)
        db.commit()
        db.refresh(product)
        return product
    
    @staticmethod
    def get_by_id(db, product_id: int) -> Optional[Product]:
        """Получение товара по ID."""
        return db.query(Product).filter(Product.id == product_id).first()
    
    @staticmethod
    def get_by_category(db, category_id: int, skip: int = 0, limit: int = 100) -> List[Product]:
        """Получение товаров по категории."""
        return db.query(Product).filter(Product.category_id == category_id).offset(skip).limit(limit).all()
    
    @staticmethod
    def search(db, query: str, skip: int = 0, limit: int = 100) -> List[Product]:
        """Поиск товаров по названию."""
        return db.query(Product).filter(Product.name.contains(query)).offset(skip).limit(limit).all()
    
    @staticmethod
    def update_stock(db, product_id: int, quantity: int, operation: str = "reduce") -> Optional[Product]:
        """Обновление количества товара."""
        product = ProductCRUD.get_by_id(db, product_id)
        if product:
            if operation == "reduce":
                product.reduce_stock(quantity)
            else:
                product.increase_stock(quantity)
            db.commit()
            db.refresh(product)
        return product


class OrderCRUD:
    """CRUD операции для модели Order."""
    
    @staticmethod
    def create(db, user_id: int) -> Order:
        """Создание заказа."""
        order = Order(user_id=user_id)
        db.add(order)
        db.commit()
        db.refresh(order)
        return order
    
    @staticmethod
    def get_by_id(db, order_id: int) -> Optional[Order]:
        """Получение заказа по ID."""
        return db.query(Order).filter(Order.id == order_id).first()
    
    @staticmethod
    def get_by_user(db, user_id: int, skip: int = 0, limit: int = 100) -> List[Order]:
        """Получение заказов пользователя."""
        return db.query(Order).filter(Order.user_id == user_id).offset(skip).limit(limit).all()
    
    @staticmethod
    def add_item(db, order_id: int, product_id: int, quantity: int = 1) -> Optional[OrderItem]:
        """Добавление товара в заказ."""
        product = ProductCRUD.get_by_id(db, product_id)
        if not product:
            return None
        
        order = OrderCRUD.get_by_id(db, order_id)
        if not order or order.status != OrderStatus.PENDING:
            return None
        
        # Проверка наличия товара
        if product.stock < quantity:
            return None
        
        # Создание элемента заказа
        order_item = OrderItem(
            order_id=order_id,
            product_id=product_id,
            quantity=quantity,
            price=product.price
        )
        db.add(order_item)
        
        # Уменьшаем количество товара
        product.reduce_stock(quantity)
        
        # Пересчитываем сумму заказа
        order.calculate_total()
        
        db.commit()
        db.refresh(order_item)
        return order_item
    
    @staticmethod
    def update_status(db, order_id: int, status: str) -> Optional[Order]:
        """Обновление статуса заказа."""
        order = OrderCRUD.get_by_id(db, order_id)
        if order and status in OrderStatus.choices():
            order.status = status
            db.commit()
            db.refresh(order)
        return order
    
    @staticmethod
    def cancel(db, order_id: int) -> bool:
        """Отмена заказа (с возвратом товаров на склад)."""
        order = OrderCRUD.get_by_id(db, order_id)
        if not order or not order.can_cancel():
            return False
        
        # Возвращаем товары на склад
        for item in order.items:
            product = ProductCRUD.get_by_id(db, item.product_id)
            if product:
                product.increase_stock(item.quantity)
        
        order.status = OrderStatus.CANCELLED
        db.commit()
        return True


# ============================================================
# СОЗДАНИЕ ТАБЛИЦ
# ============================================================

def init_db():
    """Инициализация базы данных (создание таблиц)."""
    Base.metadata.create_all(bind=engine)
    print("✅ Таблицы созданы")


# ============================================================
# ДЕМОНСТРАЦИЯ
# ============================================================

def demo():
    """Демонстрация работы с моделями."""
    
    print("=" * 60)
    print("ДЕМОНСТРАЦИЯ РАБОТЫ С МОДЕЛЯМИ ДАННЫХ")
    print("=" * 60)
    
    # Инициализация БД
    init_db()
    
    # Создание сессии
    db = SessionLocal()
    
    # 1. Создание категорий
    print("\n📁 1. Создание категорий:")
    
    electronics = Category(name="Электроника", slug="electronics", description="Электронные товары")
    books = Category(name="Книги", slug="books", description="Книжная продукция")
    
    db.add_all([electronics, books])
    db.commit()
    print(f"   Созданы категории: {electronics.name}, {books.name}")
    
    # 2. Создание товаров
    print("\n📦 2. Создание товаров:")
    
    laptop = Product(
        name="Ноутбук",
        description="Мощный ноутбук для работы",
        price=50000.00,
        stock=10,
        category=electronics
    )
    
    phone = Product(
        name="Смартфон",
        description="Современный смартфон",
        price=30000.00,
        stock=15,
        category=electronics
    )
    
    book = Product(
        name="Python для начинающих",
        description="Книга по Python",
        price=1500.00,
        stock=50,
        category=books
    )
    
    db.add_all([laptop, phone, book])
    db.commit()
    print(f"   Созданы товары: {laptop.name}, {phone.name}, {book.name}")
    
    # 3. Создание пользователя
    print("\n👤 3. Создание пользователя:")
    
    user = UserCRUD.create(
        db,
        name="Иван Петров",
        email="ivan@example.com",
        password="secret123"
    )
    print(f"   Создан пользователь: {user.name} ({user.email})")
    
    # 4. Создание профиля пользователя
    print("\n📝 4. Создание профиля:")
    
    profile = Profile(
        user_id=user.id,
        bio="Python разработчик",
        phone="+7-900-123-45-67",
        city="Москва"
    )
    db.add(profile)
    db.commit()
    print(f"   Создан профиль для {user.name}")
    
    # 5. Создание заказа
    print("\n🛒 5. Создание заказа:")
    
    order = OrderCRUD.create(db, user.id)
    print(f"   Создан заказ #{order.id}")
    
    # 6. Добавление товаров в заказ
    print("\n➕ 6. Добавление товаров в заказ:")
    
    OrderCRUD.add_item(db, order.id, laptop.id, 1)
    OrderCRUD.add_item(db, order.id, phone.id, 2)
    
    db.refresh(order)
    print(f"   Добавлено 2 товара. Сумма заказа: {order.total:.2f} ₽")
    
    # 7. Информация о заказе
    print("\n📋 7. Информация о заказе:")
    order_data = order.to_dict()
    print(f"   Заказ #{order_data['id']}")
    print(f"   Статус: {order_data['status']}")
    print(f"   Сумма: {order_data['total']} ₽")
    print("   Товары:")
    for item in order_data['items']:
        print(f"      - {item['product_name']} x{item['quantity']} = {item['subtotal']} ₽")
    
    # 8. Поиск товаров
    print("\n🔍 8. Поиск товаров:")
    search_results = ProductCRUD.search(db, "Python")
    print(f"   Результаты поиска 'Python': {len(search_results)}")
    for product in search_results:
        print(f"      - {product.name} ({product.price} ₽)")
    
    # 9. Статистика
    print("\n📊 9. Статистика:")
    users_count = db.query(User).count()
    products_count = db.query(Product).count()
    orders_count = db.query(Order).count()
    print(f"   Пользователей: {users_count}")
    print(f"   Товаров: {products_count}")
    print(f"   Заказов: {orders_count}")
    
    # Закрытие сессии
    db.close()
    
    print("\n" + "=" * 60)
    print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
    print("=" * 60)


if __name__ == "__main__":
    demo()
Ожидаемый вывод
text
============================================================
ДЕМОНСТРАЦИЯ РАБОТЫ С МОДЕЛЯМИ ДАННЫХ
============================================================

📁 1. Создание категорий:
   Созданы категории: Электроника, Книги

📦 2. Создание товаров:
   Созданы товары: Ноутбук, Смартфон, Python для начинающих

👤 3. Создание пользователя:
   Создан пользователь: Иван Петров (ivan@example.com)

📝 4. Создание профиля:
   Создан профиль для Иван Петров

🛒 5. Создание заказа:
   Создан заказ #1

➕ 6. Добавление товаров в заказ:
   Добавлено 2 товара. Сумма заказа: 110000.00 ₽

📋 7. Информация о заказе:
   Заказ #1
   Статус: pending
   Сумма: 110000.0 ₽
   Товары:
      - Ноутбук x1 = 50000.0 ₽
      - Смартфон x2 = 60000.0 ₽

🔍 8. Поиск товаров:
   Результаты поиска 'Python': 1
      - Python для начинающих (1500.0 ₽)

📊 9. Статистика:
   Пользователей: 1
   Товаров: 3
   Заказов: 1

============================================================
ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА
============================================================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (2-3 модели, простые связи)

Варианты 9-17: средний (4-5 моделей, один-ко-многим)

Варианты 18-25: сложный (5+ моделей, многие-ко-многим, наследование)

Варианты 1-8 (Базовый уровень)
№	Модели	Связи
1	User, Post	User → Post (один-ко-многим)
2	Author, Book	Author → Book (один-ко-многим)
3	Student, Grade	Student → Grade (один-ко-многим)
4	Category, Product	Category → Product (один-ко-многим)
5	Employee, Department	Department → Employee (один-ко-многим)
6	Playlist, Song	Playlist → Song (один-ко-многим)
7	Customer, Order	Customer → Order (один-ко-многим)
8	Team, Player	Team → Player (один-ко-многим)
Пример варианта 1:

python
class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String(100))
    posts = relationship("Post", back_populates="author")

class Post(Base):
    __tablename__ = 'posts'
    id = Column(Integer, primary_key=True)
    title = Column(String(200))
    user_id = Column(Integer, ForeignKey('users.id'))
    author = relationship("User", back_populates="posts")
Варианты 9-17 (Средний уровень)
№	Модели	Дополнительные требования
9	User, Post, Comment	Связи: User → Post, Post → Comment
10	Author, Book, Publisher	Связи: Author ↔ Book (многие-ко-многим)
11	Student, Course, Enrollment	Связи: Student ↔ Course через Enrollment
12	Doctor, Patient, Appointment	Связи: Doctor ↔ Patient через Appointment
13	Hotel, Room, Booking	Связи: Hotel → Room, Booking ↔ Room
14	Restaurant, Menu, Order	Связи: Restaurant → Menu, Order ↔ Menu
15	Library, Book, Reader	Связи: Book ↔ Reader через Loan
16	Event, Ticket, Attendee	Связи: Event → Ticket, Ticket → Attendee
17	Project, Task, Employee	Связи: Project → Task, Employee ↔ Task
Пример варианта 10:

python
class Author(Base):
    __tablename__ = 'authors'
    id = Column(Integer, primary_key=True)
    name = Column(String(100))
    books = relationship("Book", secondary="book_authors", back_populates="authors")

class Book(Base):
    __tablename__ = 'books'
    id = Column(Integer, primary_key=True)
    title = Column(String(200))
    authors = relationship("Author", secondary="book_authors", back_populates="books")

class BookAuthor(Base):
    __tablename__ = 'book_authors'
    book_id = Column(Integer, ForeignKey('books.id'), primary_key=True)
    author_id = Column(Integer, ForeignKey('authors.id'), primary_key=True)
Варианты 18-25 (Сложный уровень)
№	Тема	Особенности
18	Интернет-магазин	User, Profile, Category, Product, Order, OrderItem
19	Социальная сеть	User, Post, Comment, Like, Friend
20	Система бронирования	User, Hotel, Room, Booking, Payment
21	Образовательная платформа	User, Course, Lesson, Enrollment, Progress
22	Доставка еды	User, Restaurant, Menu, Cart, Order, Delivery
23	CRM система	User, Customer, Deal, Task, Activity
24	Блог с тегами	User, Post, Tag, Comment, Category
25	Спортивная лига	League, Team, Player, Match, Score
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Модели не созданы или не работают
3 (удовлетворительно)	Созданы 2-3 модели с базовыми связями
4 (хорошо)	Созданы 4-5 моделей, все связи настроены
5 (отлично)	Полная модель (6+ таблиц), связи, индексы, CRUD операции
5. Шпаргалка
python
# === БАЗОВАЯ МОДЕЛЬ ===
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Model(Base):
    __tablename__ = 'table_name'
    id = Column(Integer, primary_key=True)
    name = Column(String(100))

# === СВЯЗЬ ОДИН-КО-МНОГИМ ===
parent_id = Column(Integer, ForeignKey('parent.id'))
parent = relationship("Parent", back_populates="children")
children = relationship("Child", back_populates="parent")

# === СВЯЗЬ МНОГИЕ-КО-МНОГИМ ===
# Ассоциативная таблица
association = Table('association', Base.metadata,
    Column('left_id', Integer, ForeignKey('left.id')),
    Column('right_id', Integer, ForeignKey('right.id'))
)

# В моделях
lefts = relationship("Right", secondary=association, back_populates="lefts")

# === CRUD ===
db.add(instance)
db.commit()
db.query(Model).filter(Model.id == id).first()
db.query(Model).all()
db.delete(instance)
Карточка студента
text
ПЗ 2.28. СОЗДАНИЕ МОДЕЛЕЙ ДАННЫХ ДЛЯ API

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== МОДЕЛИ ===

1. _____________ (таблица: _____________)
   Поля: _________________________________

2. _____________ (таблица: _____________)
   Поля: _________________________________

3. _____________ (таблица: _____________)
   Поля: _________________________________

4. _____________ (таблица: _____________)
   Поля: _________________________________

=== СВЯЗИ ===

□ Один-к-одному: _____________ → _____________
□ Один-ко-многим: _____________ → _____________
□ Многие-ко-многим: _____________ ↔ _____________

=== ОТЧЁТ ===

Файл моделей: _____________
Скриншот схемы БД: _____________
Примеры CRUD операций: _____________

Дата выполнения: _____________


text
