Вни# ПЗ 2.29. Реализация CRUD-операций через SQLAlchemy

**Тема:** Работа с базами данных, SQLAlchemy ORM, CRUD операции, сессии

**Цель работы:**  
Научиться реализовывать полноценные CRUD-операции (Create, Read, Update, Delete) с использованием SQLAlchemy ORM, работать с сессиями, выполнять сложные запросы и оптимизировать взаимодействие с базой данных.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `sqlalchemy`, `alembic`, `passlib`

```bash
pip install sqlalchemy alembic passlib
Главная мысль: SQLAlchemy ORM превращает строки SQL в объекты Python. CRUD операции становятся интуитивно понятными, а код — чище и безопаснее.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. CRUD операции и SQLAlchemy
Операция	SQL запрос	SQLAlchemy ORM
Create	INSERT INTO users (name) VALUES ('Иван')	session.add(user)
Read	SELECT * FROM users WHERE id = 1	session.query(User).get(1)
Update	UPDATE users SET name = 'Петр' WHERE id = 1	user.name = "Петр"
Delete	DELETE FROM users WHERE id = 1	session.delete(user)
1.2. Жизненный цикл объекта в SQLAlchemy
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ЖИЗНЕННЫЙ ЦИКЛ ОБЪЕКТА                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────┐     session.add()    ┌─────────┐    session.commit()   ┌─────────┐
│   │ Transient│ ─────────────────► │ Pending │ ──────────────────► │  Persistent │
│   │ (новый)  │                     │(ожидает)│                       │(в БД)      │
│   └─────────┘                     └─────────┘                       └─────┬───┘
│                                                                           │
│                                              session.delete()            │
│                                                    │                      │
│                                                    ▼                      │
│                                              ┌─────────┐                 │
│                                              │ Detached │                 │
│                                              │(отсоединён)│                 │
│                                              └─────────┘                 │
└─────────────────────────────────────────────────────────────────────────────┘
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая реализацию CRUD-операций через SQLAlchemy.

Техническое задание (нулевой вариант)
Реализуйте класс DatabaseManager для выполнения CRUD-операций над моделями User, Product, Order. Используйте SQLAlchemy ORM. Поддержите:

Создание записей

Чтение (одной записи, всех записей, с фильтрацией)

Обновление записей

Удаление записей

Поиск с пагинацией

Агрегационные запросы

Эталонная реализация
python
#!/usr/bin/env python3
"""
crud_operations.py — Реализация CRUD-операций через SQLAlchemy.
"""

from sqlalchemy import (
    create_engine, Column, Integer, String, Float, Boolean, 
    DateTime, Text, ForeignKey, Numeric, func, and_, or_
)
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship
from datetime import datetime
from typing import List, Optional, Dict, Any, Tuple
from contextlib import contextmanager

# ============================================================
# НАСТРОЙКА БАЗЫ ДАННЫХ
# ============================================================

DATABASE_URL = "sqlite:///./crud_demo.db"
engine = create_engine(DATABASE_URL, echo=True)
Base = declarative_base()
SessionLocal = sessionmaker(bind=engine)


@contextmanager
def get_db():
    """Контекстный менеджер для работы с сессией."""
    session = SessionLocal()
    try:
        yield session
        session.commit()
    except Exception as e:
        session.rollback()
        raise e
    finally:
        session.close()


# ============================================================
# МОДЕЛИ ДАННЫХ
# ============================================================

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True, nullable=False, index=True)
    age = Column(Integer, default=0)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    orders = relationship("Order", back_populates="user", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<User(id={self.id}, name={self.name}, email={self.email})>"


class Product(Base):
    __tablename__ = "products"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(200), nullable=False, index=True)
    description = Column(Text, nullable=True)
    price = Column(Numeric(10, 2), nullable=False)
    stock = Column(Integer, default=0)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    order_items = relationship("OrderItem", back_populates="product")
    
    def __repr__(self):
        return f"<Product(id={self.id}, name={self.name}, price={self.price})>"


class Order(Base):
    __tablename__ = "orders"
    
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    status = Column(String(20), default="pending")
    total = Column(Numeric(10, 2), default=0.00)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
    
    user = relationship("User", back_populates="orders")
    items = relationship("OrderItem", back_populates="order", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<Order(id={self.id}, user_id={self.user_id}, total={self.total})>"


class OrderItem(Base):
    __tablename__ = "order_items"
    
    id = Column(Integer, primary_key=True, index=True)
    order_id = Column(Integer, ForeignKey("orders.id"), nullable=False)
    product_id = Column(Integer, ForeignKey("products.id"), nullable=False)
    quantity = Column(Integer, nullable=False, default=1)
    price = Column(Numeric(10, 2), nullable=False)
    
    order = relationship("Order", back_populates="items")
    product = relationship("Product", back_populates="order_items")


# Создание таблиц
Base.metadata.create_all(bind=engine)


# ============================================================
# БАЗОВЫЙ CRUD КЛАСС
# ============================================================

class BaseCRUD:
    """Базовый класс для CRUD-операций."""
    
    model = None
    
    @classmethod
    def create(cls, session, **kwargs):
        """Создание записи."""
        instance = cls.model(**kwargs)
        session.add(instance)
        session.flush()  # Получаем ID без коммита
        return instance
    
    @classmethod
    def get_by_id(cls, session, id: int):
        """Получение записи по ID."""
        return session.query(cls.model).filter(cls.model.id == id).first()
    
    @classmethod
    def get_all(cls, session, skip: int = 0, limit: int = 100):
        """Получение всех записей с пагинацией."""
        return session.query(cls.model).offset(skip).limit(limit).all()
    
    @classmethod
    def update(cls, session, id: int, **kwargs):
        """Обновление записи."""
        instance = cls.get_by_id(session, id)
        if instance:
            for key, value in kwargs.items():
                if hasattr(instance, key):
                    setattr(instance, key, value)
            session.flush()
        return instance
    
    @classmethod
    def delete(cls, session, id: int) -> bool:
        """Удаление записи."""
        instance = cls.get_by_id(session, id)
        if instance:
            session.delete(instance)
            session.flush()
            return True
        return False
    
    @classmethod
    def count(cls, session, **filters) -> int:
        """Подсчёт количества записей."""
        query = session.query(cls.model)
        for key, value in filters.items():
            if hasattr(cls.model, key):
                query = query.filter(getattr(cls.model, key) == value)
        return query.count()
    
    @classmethod
    def exists(cls, session, **filters) -> bool:
        """Проверка существования записи."""
        return cls.count(session, **filters) > 0


# ============================================================
# USER CRUD
# ============================================================

class UserCRUD(BaseCRUD):
    """CRUD операции для модели User."""
    
    model = User
    
    @classmethod
    def get_by_email(cls, session, email: str) -> Optional[User]:
        """Получение пользователя по email."""
        return session.query(User).filter(User.email == email).first()
    
    @classmethod
    def get_active_users(cls, session, skip: int = 0, limit: int = 100) -> List[User]:
        """Получение активных пользователей."""
        return session.query(User).filter(User.is_active == True).offset(skip).limit(limit).all()
    
    @classmethod
    def search_by_name(cls, session, query: str, skip: int = 0, limit: int = 100) -> List[User]:
        """Поиск пользователей по имени."""
        return session.query(User).filter(User.name.contains(query)).offset(skip).limit(limit).all()
    
    @classmethod
    def get_users_by_age_range(cls, session, min_age: int, max_age: int) -> List[User]:
        """Получение пользователей в диапазоне возраста."""
        return session.query(User).filter(and_(User.age >= min_age, User.age <= max_age)).all()
    
    @classmethod
    def bulk_create(cls, session, users_data: List[Dict]) -> List[User]:
        """Массовое создание пользователей."""
        users = [cls.model(**data) for data in users_data]
        session.add_all(users)
        session.flush()
        return users
    
    @classmethod
    def bulk_update_active(cls, session, user_ids: List[int], is_active: bool) -> int:
        """Массовое обновление статуса активности."""
        updated = session.query(User).filter(User.id.in_(user_ids)).update(
            {User.is_active: is_active},
            synchronize_session=False
        )
        return updated
    
    @classmethod
    def bulk_delete_inactive(cls, session) -> int:
        """Массовое удаление неактивных пользователей."""
        deleted = session.query(User).filter(User.is_active == False).delete()
        return deleted
    
    @classmethod
    def get_statistics(cls, session) -> Dict:
        """Статистика по пользователям."""
        stats = session.query(
            func.count(User.id).label('total'),
            func.sum(User.is_active.cast(Integer)).label('active'),
            func.avg(User.age).label('avg_age'),
            func.min(User.age).label('min_age'),
            func.max(User.age).label('max_age')
        ).first()
        
        return {
            "total": stats.total or 0,
            "active": stats.active or 0,
            "inactive": (stats.total or 0) - (stats.active or 0),
            "avg_age": round(stats.avg_age, 1) if stats.avg_age else 0,
            "min_age": stats.min_age or 0,
            "max_age": stats.max_age or 0
        }


# ============================================================
# PRODUCT CRUD
# ============================================================

class ProductCRUD(BaseCRUD):
    """CRUD операции для модели Product."""
    
    model = Product
    
    @classmethod
    def get_by_name(cls, session, name: str) -> Optional[Product]:
        """Получение товара по названию."""
        return session.query(Product).filter(Product.name == name).first()
    
    @classmethod
    def get_active_products(cls, session, skip: int = 0, limit: int = 100) -> List[Product]:
        """Получение активных товаров."""
        return session.query(Product).filter(Product.is_active == True).offset(skip).limit(limit).all()
    
    @classmethod
    def get_products_by_price_range(cls, session, min_price: float, max_price: float) -> List[Product]:
        """Получение товаров в диапазоне цен."""
        return session.query(Product).filter(
            and_(Product.price >= min_price, Product.price <= max_price)
        ).all()
    
    @classmethod
    def get_low_stock_products(cls, session, threshold: int = 10) -> List[Product]:
        """Получение товаров с низким остатком."""
        return session.query(Product).filter(Product.stock <= threshold).all()
    
    @classmethod
    def search_products(cls, session, query: str, skip: int = 0, limit: int = 100) -> List[Product]:
        """Поиск товаров по названию и описанию."""
        return session.query(Product).filter(
            or_(
                Product.name.contains(query),
                Product.description.contains(query)
            )
        ).offset(skip).limit(limit).all()
    
    @classmethod
    def update_stock(cls, session, product_id: int, quantity: int, operation: str = "reduce") -> Optional[Product]:
        """Обновление количества товара."""
        product = cls.get_by_id(session, product_id)
        if product:
            if operation == "reduce":
                product.stock -= quantity
            elif operation == "increase":
                product.stock += quantity
            elif operation == "set":
                product.stock = quantity
            session.flush()
        return product


# ============================================================
# ORDER CRUD
# ============================================================

class OrderCRUD(BaseCRUD):
    """CRUD операции для модели Order."""
    
    model = Order
    
    @classmethod
    def create_order(cls, session, user_id: int) -> Order:
        """Создание заказа."""
        return cls.create(session, user_id=user_id)
    
    @classmethod
    def get_orders_by_user(cls, session, user_id: int, skip: int = 0, limit: int = 100) -> List[Order]:
        """Получение заказов пользователя."""
        return session.query(Order).filter(Order.user_id == user_id).offset(skip).limit(limit).all()
    
    @classmethod
    def get_orders_by_status(cls, session, status: str) -> List[Order]:
        """Получение заказов по статусу."""
        return session.query(Order).filter(Order.status == status).all()
    
    @classmethod
    def add_item(cls, session, order_id: int, product_id: int, quantity: int = 1) -> Optional[OrderItem]:
        """Добавление товара в заказ."""
        product = ProductCRUD.get_by_id(session, product_id)
        if not product:
            return None
        
        order = cls.get_by_id(session, order_id)
        if not order or order.status != "pending":
            return None
        
        if product.stock < quantity:
            return None
        
        # Создание элемента заказа
        order_item = OrderItem(
            order_id=order_id,
            product_id=product_id,
            quantity=quantity,
            price=product.price
        )
        session.add(order_item)
        
        # Уменьшаем количество товара
        product.stock -= quantity
        
        # Пересчитываем сумму заказа
        cls._recalculate_total(session, order_id)
        
        session.flush()
        return order_item
    
    @classmethod
    def _recalculate_total(cls, session, order_id: int):
        """Пересчёт суммы заказа."""
        order = cls.get_by_id(session, order_id)
        if order:
            total = session.query(func.sum(OrderItem.price * OrderItem.quantity)).filter(
                OrderItem.order_id == order_id
            ).scalar()
            order.total = total or 0.00
            session.flush()
    
    @classmethod
    def update_status(cls, session, order_id: int, status: str) -> Optional[Order]:
        """Обновление статуса заказа."""
        valid_statuses = ["pending", "processing", "shipped", "delivered", "cancelled"]
        if status not in valid_statuses:
            return None
        
        order = cls.get_by_id(session, order_id)
        if order:
            order.status = status
            session.flush()
        return order
    
    @classmethod
    def cancel_order(cls, session, order_id: int) -> bool:
        """Отмена заказа (с возвратом товаров)."""
        order = cls.get_by_id(session, order_id)
        if not order or order.status not in ["pending", "processing"]:
            return False
        
        # Возвращаем товары на склад
        for item in order.items:
            product = ProductCRUD.get_by_id(session, item.product_id)
            if product:
                product.stock += item.quantity
        
        order.status = "cancelled"
        session.flush()
        return True
    
    @classmethod
    def get_order_with_items(cls, session, order_id: int) -> Optional[Dict]:
        """Получение заказа с товарами."""
        order = cls.get_by_id(session, order_id)
        if not order:
            return None
        
        return {
            "id": order.id,
            "user_id": order.user_id,
            "status": order.status,
            "total": float(order.total),
            "created_at": order.created_at.isoformat() if order.created_at else None,
            "items": [
                {
                    "product_id": item.product_id,
                    "product_name": item.product.name,
                    "quantity": item.quantity,
                    "price": float(item.price),
                    "subtotal": float(item.price * item.quantity)
                }
                for item in order.items
            ]
        }


# ============================================================
# ДЕМОНСТРАЦИЯ
# ============================================================

def main():
    """Демонстрация CRUD операций."""
    
    print("=" * 70)
    print("ДЕМОНСТРАЦИЯ CRUD ОПЕРАЦИЙ ЧЕРЕЗ SQLALCHEMY")
    print("=" * 70)
    
    with get_db() as session:
        
        # ========== CREATE ==========
        print("\n📝 1. CREATE — Создание записей")
        print("-" * 50)
        
        # Создание пользователей
        user1 = UserCRUD.create(session, name="Иван Петров", email="ivan@example.com", age=25)
        user2 = UserCRUD.create(session, name="Мария Сидорова", email="maria@example.com", age=30)
        user3 = UserCRUD.create(session, name="Петр Иванов", email="petr@example.com", age=20)
        
        print(f"   Созданы пользователи: {user1.name}, {user2.name}, {user3.name}")
        
        # Массовое создание
        users_data = [
            {"name": "Анна", "email": "anna@example.com", "age": 28},
            {"name": "Дмитрий", "email": "dmitry@example.com", "age": 35}
        ]
        new_users = UserCRUD.bulk_create(session, users_data)
        print(f"   Массовое создание: {len(new_users)} пользователей")
        
        # Создание товаров
        product1 = ProductCRUD.create(session, name="Ноутбук", price=50000, stock=10)
        product2 = ProductCRUD.create(session, name="Мышь", price=1500, stock=50)
        product3 = ProductCRUD.create(session, name="Клавиатура", price=3000, stock=30)
        
        print(f"   Созданы товары: {product1.name}, {product2.name}, {product3.name}")
        
        # ========== READ ==========
        print("\n📖 2. READ — Чтение данных")
        print("-" * 50)
        
        # Получение по ID
        user = UserCRUD.get_by_id(session, user1.id)
        print(f"   get_by_id(1): {user.name}")
        
        # Получение по email
        user = UserCRUD.get_by_email(session, "maria@example.com")
        print(f"   get_by_email('maria@example.com'): {user.name}")
        
        # Получение всех с пагинацией
        users = UserCRUD.get_all(session, skip=0, limit=10)
        print(f"   get_all(): {len(users)} пользователей")
        
        # Активные пользователи
        active_users = UserCRUD.get_active_users(session)
        print(f"   get_active_users(): {len(active_users)}")
        
        # Поиск по имени
        search_results = UserCRUD.search_by_name(session, "Иван")
        print(f"   search_by_name('Иван'): {len(search_results)}")
        
        # Товары в диапазоне цен
        products = ProductCRUD.get_products_by_price_range(session, 1000, 10000)
        print(f"   get_products_by_price_range(1000-10000): {len(products)} товаров")
        
        # Поиск товаров
        search_products = ProductCRUD.search_products(session, "ноут")
        print(f"   search_products('ноут'): {len(search_products)}")
        
        # ========== UPDATE ==========
        print("\n✏️ 3. UPDATE — Обновление данных")
        print("-" * 50)
        
        # Обновление пользователя
        updated = UserCRUD.update(session, user1.id, age=26, name="Иван Сергеевич Петров")
        print(f"   Обновлён пользователь: {updated.name} (возраст: {updated.age})")
        
        # Массовое обновление статуса
        user_ids = [user1.id, user2.id]
        count = UserCRUD.bulk_update_active(session, user_ids, True)
        print(f"   Массовое обновление активности: {count} пользователей")
        
        # Обновление товара
        ProductCRUD.update(session, product1.id, price=45000)
        print(f"   Обновлён товар: {product1.name} (новая цена: {product1.price})")
        
        # Обновление остатка
        ProductCRUD.update_stock(session, product1.id, 2, "reduce")
        print(f"   Обновлён остаток: {product1.name} (остаток: {product1.stock})")
        
        # ========== DELETE ==========
        print("\n🗑️ 4. DELETE — Удаление данных")
        print("-" * 50)
        
        # Удаление пользователя (создадим временного)
        temp_user = UserCRUD.create(session, name="Temp", email="temp@example.com", age=99)
        deleted = UserCRUD.delete(session, temp_user.id)
        print(f"   Удаление пользователя: {'успешно' if deleted else 'неудачно'}")
        
        # ========== СТАТИСТИКА ==========
        print("\n📊 5. СТАТИСТИКА")
        print("-" * 50)
        
        stats = UserCRUD.get_statistics(session)
        print(f"   Пользователи:")
        print(f"      Всего: {stats['total']}")
        print(f"      Активных: {stats['active']}")
        print(f"      Средний возраст: {stats['avg_age']}")
        print(f"      Возраст: {stats['min_age']} - {stats['max_age']}")
        
        product_stats = session.query(
            func.count(Product.id).label('total'),
            func.sum(Product.stock).label('total_stock'),
            func.avg(Product.price).label('avg_price')
        ).first()
        
        print(f"\n   Товары:")
        print(f"      Всего: {product_stats.total}")
        print(f"      Общий остаток: {product_stats.total_stock}")
        print(f"      Средняя цена: {product_stats.avg_price:.2f}")
        
        # ========== ORDER ==========
        print("\n🛒 6. РАБОТА С ЗАКАЗАМИ")
        print("-" * 50)
        
        # Создание заказа
        order = OrderCRUD.create_order(session, user1.id)
        print(f"   Создан заказ #{order.id}")
        
        # Добавление товаров
        item1 = OrderCRUD.add_item(session, order.id, product1.id, 1)
        item2 = OrderCRUD.add_item(session, order.id, product2.id, 2)
        print(f"   Добавлены товары: {product1.name} x1, {product2.name} x2")
        
        # Получение заказа с товарами
        order_data = OrderCRUD.get_order_with_items(session, order.id)
        if order_data:
            print(f"   Заказ #{order_data['id']}: сумма = {order_data['total']}")
            for item in order_data['items']:
                print(f"      - {item['product_name']} x{item['quantity']} = {item['subtotal']}")
        
        # Изменение статуса
        OrderCRUD.update_status(session, order.id, "processing")
        print(f"   Статус заказа изменён на: processing")
        
        # ========== ПАГИНАЦИЯ ==========
        print("\n📄 7. ПАГИНАЦИЯ")
        print("-" * 50)
        
        page_size = 2
        for page in range(1, 4):
            users = UserCRUD.get_all(session, skip=(page-1)*page_size, limit=page_size)
            print(f"   Страница {page}: {len(users)} пользователей")
            for u in users:
                print(f"      - {u.name}")
        
        print("\n" + "=" * 70)
        print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
        print("=" * 70)


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
======================================================================
ДЕМОНСТРАЦИЯ CRUD ОПЕРАЦИЙ ЧЕРЕЗ SQLALCHEMY
======================================================================

📝 1. CREATE — Создание записій
--------------------------------------------------
   Созданы пользователи: Иван Петров, Мария Сидорова, Петр Иванов
   Массовое создание: 2 пользователей
   Созданы товары: Ноутбук, Мышь, Клавиатура

📖 2. READ — Чтение данных
--------------------------------------------------
   get_by_id(1): Иван Петров
   get_by_email('maria@example.com'): Мария Сидорова
   get_all(): 5 пользователей
   get_active_users(): 5
   search_by_name('Иван'): 1
   get_products_by_price_range(1000-10000): 2 товаров
   search_products('ноут'): 1

✏️ 3. UPDATE — Обновление данных
--------------------------------------------------
   Обновлён пользователь: Иван Сергеевич Петров (возраст: 26)
   Массовое обновление активности: 2 пользователей
   Обновлён товар: Ноутбук (новая цена: 45000)
   Обновлён остаток: Ноутбук (остаток: 8)

🗑️ 4. DELETE — Удаление данных
--------------------------------------------------
   Удаление пользователя: успешно

📊 5. СТАТИСТИКА
--------------------------------------------------
   Пользователи:
      Всего: 5
      Активных: 5
      Средний возраст: 27.6
      Возраст: 20 - 35

   Товары:
      Всего: 3
      Общий остаток: 88
      Средняя цена: 16500.00

🛒 6. РАБОТА С ЗАКАЗАМИ
--------------------------------------------------
   Создан заказ #1
   Добавлены товары: Ноутбук x1, Мышь x2
   Заказ #1: сумма = 48000.0
      - Ноутбук x1 = 45000.0
      - Мышь x2 = 3000.0
   Статус заказа изменён на: processing

📄 7. ПАГИНАЦИЯ
--------------------------------------------------
   Страница 1: 2 пользователей
      - Иван Сергеевич Петров
      - Мария Сидорова
   Страница 2: 2 пользователей
      - Петр Иванов
      - Анна
   Страница 3: 1 пользователей
      - Дмитрий

======================================================================
ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА
======================================================================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (CRUD для одной модели)

Варианты 9-17: средний (CRUD + связи)

Варианты 18-25: сложный (CRUD + агрегации + транзакции)

Варианты 1-8 (Базовый уровень)
№	Модель	Поля	CRUD операции
1	User	name, email, age	✅
2	Product	name, price, stock	✅
3	Post	title, content, author	✅
4	Task	title, description, completed	✅
5	Book	title, author, year	✅
6	Movie	title, director, rating	✅
7	Student	name, group, score	✅
8	Employee	name, position, salary	✅
Варианты 9-17 (Средний уровень)
№	Модели	CRUD операции	Связи
9	User, Post	✅ + поиск по автору	User → Post
10	Author, Book	✅ + поиск по автору	Author → Book
11	Category, Product	✅ + фильтрация по категории	Category → Product
12	Customer, Order	✅ + расчёт суммы заказа	Customer → Order
13	Doctor, Appointment	✅ + фильтрация по дате	Doctor → Appointment
14	Playlist, Song	✅ + поиск по плейлисту	Playlist ↔ Song
15	Team, Player	✅ + статистика команды	Team → Player
16	Event, Ticket	✅ + проверка наличия	Event → Ticket
17	Article, Comment	✅ + подсчёт комментариев	Article → Comment
Варианты 18-25 (Сложный уровень)
№	Тема	Особенности
18	Интернет-магазин	User, Product, Order, OrderItem + отчёты
19	Блог	User, Post, Comment, Tag + агрегации
20	Библиотека	User, Book, Loan, Author + штрафы
21	CRM	User, Customer, Deal, Activity + воронка
22	Образование	Student, Course, Teacher, Grade + успеваемость
23	Доставка	User, Order, Courier, Delivery + трекинг
24	Финансы	User, Account, Transaction, Category + баланс
25	Склад	Product, Supplier, Warehouse, Stock + резервирование
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	CRUD не работает или код не запускается
3 (удовлетворительно)	Реализованы базовые CRUD (create, read, update, delete)
4 (хорошо)	+ поиск, фильтрация, пагинация
5 (отлично)	+ массовые операции, агрегации, транзакции
5. Шпаргалка
python
# === БАЗОВЫЙ CRUD ===
session.add(instance)           # CREATE
session.query(Model).get(id)    # READ (один)
session.query(Model).all()      # READ (все)
session.query(Model).filter(...)# READ (фильтр)
instance.attr = value           # UPDATE
session.delete(instance)        # DELETE
session.commit()                # Фиксация
session.rollback()              # Откат

# === ФИЛЬТРАЦИЯ ===
Model.name == "value"           # Равенство
Model.age > 18                  # Больше
Model.age.between(18, 65)       # Диапазон
Model.name.contains("text")     # Содержит
Model.name.in_(['a', 'b'])      # В списке
and_(cond1, cond2)              # И
or_(cond1, cond2)               # ИЛИ

# === АГРЕГАЦИИ ===
func.count(Model.id)            # Количество
func.sum(Model.price)           # Сумма
func.avg(Model.age)             # Среднее
func.min(Model.age)             # Минимум
func.max(Model.age)             # Максимум

# === МАССОВЫЕ ОПЕРАЦИИ ===
session.add_all([obj1, obj2])   # Массовое добавление
session.query(Model).update(...)# Массовое обновление
session.query(Model).delete()   # Массовое удаление
Карточка студента
text
ПЗ 2.29. РЕАЛИЗАЦИЯ CRUD-ОПЕРАЦИЙ ЧЕРЕЗ SQLALCHEMY

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== CRUD ОПЕРАЦИИ ===

мательнее -> Исключительно в формате Markdown ->
                                   # ПЗ 2.29. Реализация CRUD-операций через SQLAlchemy

**Тема:** Работа с базами данных, SQLAlchemy ORM, CRUD операции, сессии

**Цель работы:**  
Научиться реализовывать полноценные CRUD-операции (Create, Read, Update, Delete) с использованием SQLAlchemy ORM, работать с сессиями, выполнять сложные запросы и оптимизировать взаимодействие с базой данных.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `sqlalchemy`, `alembic`, `passlib`

```bash
pip install sqlalchemy alembic passlib
Главная мысль: SQLAlchemy ORM превращает строки SQL в объекты Python. CRUD операции становятся интуитивно понятными, а код — чище и безопаснее.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. CRUD операции и SQLAlchemy
Операция	SQL запрос	SQLAlchemy ORM
Create	INSERT INTO users (name) VALUES ('Иван')	session.add(user)
Read	SELECT * FROM users WHERE id = 1	session.query(User).get(1)
Update	UPDATE users SET name = 'Петр' WHERE id = 1	user.name = "Петр"
Delete	DELETE FROM users WHERE id = 1	session.delete(user)
1.2. Жизненный цикл объекта в SQLAlchemy
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ЖИЗНЕННЫЙ ЦИКЛ ОБЪЕКТА                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────┐     session.add()    ┌─────────┐    session.commit()   ┌─────────┐
│   │ Transient│ ─────────────────► │ Pending │ ──────────────────► │  Persistent │
│   │ (новый)  │                     │(ожидает)│                       │(в БД)      │
│   └─────────┘                     └─────────┘                       └─────┬───┘
│                                                                           │
│                                              session.delete()            │
│                                                    │                      │
│                                                    ▼                      │
│                                              ┌─────────┐                 │
│                                              │ Detached │                 │
│                                              │(отсоединён)│                 │
│                                              └─────────┘                 │
└─────────────────────────────────────────────────────────────────────────────┘
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая реализацию CRUD-операций через SQLAlchemy.

Техническое задание (нулевой вариант)
Реализуйте класс DatabaseManager для выполнения CRUD-операций над моделями User, Product, Order. Используйте SQLAlchemy ORM. Поддержите:

Создание записей

Чтение (одной записи, всех записей, с фильтрацией)

Обновление записей

Удаление записей

Поиск с пагинацией

Агрегационные запросы

Эталонная реализация
python
#!/usr/bin/env python3
"""
crud_operations.py — Реализация CRUD-операций через SQLAlchemy.
"""

from sqlalchemy import (
    create_engine, Column, Integer, String, Float, Boolean, 
    DateTime, Text, ForeignKey, Numeric, func, and_, or_
)
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship
from datetime import datetime
from typing import List, Optional, Dict, Any, Tuple
from contextlib import contextmanager

# ============================================================
# НАСТРОЙКА БАЗЫ ДАННЫХ
# ============================================================

DATABASE_URL = "sqlite:///./crud_demo.db"
engine = create_engine(DATABASE_URL, echo=True)
Base = declarative_base()
SessionLocal = sessionmaker(bind=engine)


@contextmanager
def get_db():
    """Контекстный менеджер для работы с сессией."""
    session = SessionLocal()
    try:
        yield session
        session.commit()
    except Exception as e:
        session.rollback()
        raise e
    finally:
        session.close()


# ============================================================
# МОДЕЛИ ДАННЫХ
# ============================================================

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True, nullable=False, index=True)
    age = Column(Integer, default=0)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    orders = relationship("Order", back_populates="user", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<User(id={self.id}, name={self.name}, email={self.email})>"


class Product(Base):
    __tablename__ = "products"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(200), nullable=False, index=True)
    description = Column(Text, nullable=True)
    price = Column(Numeric(10, 2), nullable=False)
    stock = Column(Integer, default=0)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    order_items = relationship("OrderItem", back_populates="product")
    
    def __repr__(self):
        return f"<Product(id={self.id}, name={self.name}, price={self.price})>"


class Order(Base):
    __tablename__ = "orders"
    
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    status = Column(String(20), default="pending")
    total = Column(Numeric(10, 2), default=0.00)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
    
    user = relationship("User", back_populates="orders")
    items = relationship("OrderItem", back_populates="order", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<Order(id={self.id}, user_id={self.user_id}, total={self.total})>"


class OrderItem(Base):
    __tablename__ = "order_items"
    
    id = Column(Integer, primary_key=True, index=True)
    order_id = Column(Integer, ForeignKey("orders.id"), nullable=False)
    product_id = Column(Integer, ForeignKey("products.id"), nullable=False)
    quantity = Column(Integer, nullable=False, default=1)
    price = Column(Numeric(10, 2), nullable=False)
    
    order = relationship("Order", back_populates="items")
    product = relationship("Product", back_populates="order_items")


# Создание таблиц
Base.metadata.create_all(bind=engine)


# ============================================================
# БАЗОВЫЙ CRUD КЛАСС
# ============================================================

class BaseCRUD:
    """Базовый класс для CRUD-операций."""
    
    model = None
    
    @classmethod
    def create(cls, session, **kwargs):
        """Создание записи."""
        instance = cls.model(**kwargs)
        session.add(instance)
        session.flush()  # Получаем ID без коммита
        return instance
    
    @classmethod
    def get_by_id(cls, session, id: int):
        """Получение записи по ID."""
        return session.query(cls.model).filter(cls.model.id == id).first()
    
    @classmethod
    def get_all(cls, session, skip: int = 0, limit: int = 100):
        """Получение всех записей с пагинацией."""
        return session.query(cls.model).offset(skip).limit(limit).all()
    
    @classmethod
    def update(cls, session, id: int, **kwargs):
        """Обновление записи."""
        instance = cls.get_by_id(session, id)
        if instance:
            for key, value in kwargs.items():
                if hasattr(instance, key):
                    setattr(instance, key, value)
            session.flush()
        return instance
    
    @classmethod
    def delete(cls, session, id: int) -> bool:
        """Удаление записи."""
        instance = cls.get_by_id(session, id)
        if instance:
            session.delete(instance)
            session.flush()
            return True
        return False
    
    @classmethod
    def count(cls, session, **filters) -> int:
        """Подсчёт количества записей."""
        query = session.query(cls.model)
        for key, value in filters.items():
            if hasattr(cls.model, key):
                query = query.filter(getattr(cls.model, key) == value)
        return query.count()
    
    @classmethod
    def exists(cls, session, **filters) -> bool:
        """Проверка существования записи."""
        return cls.count(session, **filters) > 0


# ============================================================
# USER CRUD
# ============================================================

class UserCRUD(BaseCRUD):
    """CRUD операции для модели User."""
    
    model = User
    
    @classmethod
    def get_by_email(cls, session, email: str) -> Optional[User]:
        """Получение пользователя по email."""
        return session.query(User).filter(User.email == email).first()
    
    @classmethod
    def get_active_users(cls, session, skip: int = 0, limit: int = 100) -> List[User]:
        """Получение активных пользователей."""
        return session.query(User).filter(User.is_active == True).offset(skip).limit(limit).all()
    
    @classmethod
    def search_by_name(cls, session, query: str, skip: int = 0, limit: int = 100) -> List[User]:
        """Поиск пользователей по имени."""
        return session.query(User).filter(User.name.contains(query)).offset(skip).limit(limit).all()
    
    @classmethod
    def get_users_by_age_range(cls, session, min_age: int, max_age: int) -> List[User]:
        """Получение пользователей в диапазоне возраста."""
        return session.query(User).filter(and_(User.age >= min_age, User.age <= max_age)).all()
    
    @classmethod
    def bulk_create(cls, session, users_data: List[Dict]) -> List[User]:
        """Массовое создание пользователей."""
        users = [cls.model(**data) for data in users_data]
        session.add_all(users)
        session.flush()
        return users
    
    @classmethod
    def bulk_update_active(cls, session, user_ids: List[int], is_active: bool) -> int:
        """Массовое обновление статуса активности."""
        updated = session.query(User).filter(User.id.in_(user_ids)).update(
            {User.is_active: is_active},
            synchronize_session=False
        )
        return updated
    
    @classmethod
    def bulk_delete_inactive(cls, session) -> int:
        """Массовое удаление неактивных пользователей."""
        deleted = session.query(User).filter(User.is_active == False).delete()
        return deleted
    
    @classmethod
    def get_statistics(cls, session) -> Dict:
        """Статистика по пользователям."""
        stats = session.query(
            func.count(User.id).label('total'),
            func.sum(User.is_active.cast(Integer)).label('active'),
            func.avg(User.age).label('avg_age'),
            func.min(User.age).label('min_age'),
            func.max(User.age).label('max_age')
        ).first()
        
        return {
            "total": stats.total or 0,
            "active": stats.active or 0,
            "inactive": (stats.total or 0) - (stats.active or 0),
            "avg_age": round(stats.avg_age, 1) if stats.avg_age else 0,
            "min_age": stats.min_age or 0,
            "max_age": stats.max_age or 0
        }


# ============================================================
# PRODUCT CRUD
# ============================================================

class ProductCRUD(BaseCRUD):
    """CRUD операции для модели Product."""
    
    model = Product
    
    @classmethod
    def get_by_name(cls, session, name: str) -> Optional[Product]:
        """Получение товара по названию."""
        return session.query(Product).filter(Product.name == name).first()
    
    @classmethod
    def get_active_products(cls, session, skip: int = 0, limit: int = 100) -> List[Product]:
        """Получение активных товаров."""
        return session.query(Product).filter(Product.is_active == True).offset(skip).limit(limit).all()
    
    @classmethod
    def get_products_by_price_range(cls, session, min_price: float, max_price: float) -> List[Product]:
        """Получение товаров в диапазоне цен."""
        return session.query(Product).filter(
            and_(Product.price >= min_price, Product.price <= max_price)
        ).all()
    
    @classmethod
    def get_low_stock_products(cls, session, threshold: int = 10) -> List[Product]:
        """Получение товаров с низким остатком."""
        return session.query(Product).filter(Product.stock <= threshold).all()
    
    @classmethod
    def search_products(cls, session, query: str, skip: int = 0, limit: int = 100) -> List[Product]:
        """Поиск товаров по названию и описанию."""
        return session.query(Product).filter(
            or_(
                Product.name.contains(query),
                Product.description.contains(query)
            )
        ).offset(skip).limit(limit).all()
    
    @classmethod
    def update_stock(cls, session, product_id: int, quantity: int, operation: str = "reduce") -> Optional[Product]:
        """Обновление количества товара."""
        product = cls.get_by_id(session, product_id)
        if product:
            if operation == "reduce":
                product.stock -= quantity
            elif operation == "increase":
                product.stock += quantity
            elif operation == "set":
                product.stock = quantity
            session.flush()
        return product


# ============================================================
# ORDER CRUD
# ============================================================

class OrderCRUD(BaseCRUD):
    """CRUD операции для модели Order."""
    
    model = Order
    
    @classmethod
    def create_order(cls, session, user_id: int) -> Order:
        """Создание заказа."""
        return cls.create(session, user_id=user_id)
    
    @classmethod
    def get_orders_by_user(cls, session, user_id: int, skip: int = 0, limit: int = 100) -> List[Order]:
        """Получение заказов пользователя."""
        return session.query(Order).filter(Order.user_id == user_id).offset(skip).limit(limit).all()
    
    @classmethod
    def get_orders_by_status(cls, session, status: str) -> List[Order]:
        """Получение заказов по статусу."""
        return session.query(Order).filter(Order.status == status).all()
    
    @classmethod
    def add_item(cls, session, order_id: int, product_id: int, quantity: int = 1) -> Optional[OrderItem]:
        """Добавление товара в заказ."""
        product = ProductCRUD.get_by_id(session, product_id)
        if not product:
            return None
        
        order = cls.get_by_id(session, order_id)
        if not order or order.status != "pending":
            return None
        
        if product.stock < quantity:
            return None
        
        # Создание элемента заказа
        order_item = OrderItem(
            order_id=order_id,
            product_id=product_id,
            quantity=quantity,
            price=product.price
        )
        session.add(order_item)
        
        # Уменьшаем количество товара
        product.stock -= quantity
        
        # Пересчитываем сумму заказа
        cls._recalculate_total(session, order_id)
        
        session.flush()
        return order_item
    
    @classmethod
    def _recalculate_total(cls, session, order_id: int):
        """Пересчёт суммы заказа."""
        order = cls.get_by_id(session, order_id)
        if order:
            total = session.query(func.sum(OrderItem.price * OrderItem.quantity)).filter(
                OrderItem.order_id == order_id
            ).scalar()
            order.total = total or 0.00
            session.flush()
    
    @classmethod
    def update_status(cls, session, order_id: int, status: str) -> Optional[Order]:
        """Обновление статуса заказа."""
        valid_statuses = ["pending", "processing", "shipped", "delivered", "cancelled"]
        if status not in valid_statuses:
            return None
        
        order = cls.get_by_id(session, order_id)
        if order:
            order.status = status
            session.flush()
        return order
    
    @classmethod
    def cancel_order(cls, session, order_id: int) -> bool:
        """Отмена заказа (с возвратом товаров)."""
        order = cls.get_by_id(session, order_id)
        if not order or order.status not in ["pending", "processing"]:
            return False
        
        # Возвращаем товары на склад
        for item in order.items:
            product = ProductCRUD.get_by_id(session, item.product_id)
            if product:
                product.stock += item.quantity
        
        order.status = "cancelled"
        session.flush()
        return True
    
    @classmethod
    def get_order_with_items(cls, session, order_id: int) -> Optional[Dict]:
        """Получение заказа с товарами."""
        order = cls.get_by_id(session, order_id)
        if not order:
            return None
        
        return {
            "id": order.id,
            "user_id": order.user_id,
            "status": order.status,
            "total": float(order.total),
            "created_at": order.created_at.isoformat() if order.created_at else None,
            "items": [
                {
                    "product_id": item.product_id,
                    "product_name": item.product.name,
                    "quantity": item.quantity,
                    "price": float(item.price),
                    "subtotal": float(item.price * item.quantity)
                }
                for item in order.items
            ]
        }


# ============================================================
# ДЕМОНСТРАЦИЯ
# ============================================================

def main():
    """Демонстрация CRUD операций."""
    
    print("=" * 70)
    print("ДЕМОНСТРАЦИЯ CRUD ОПЕРАЦИЙ ЧЕРЕЗ SQLALCHEMY")
    print("=" * 70)
    
    with get_db() as session:
        
        # ========== CREATE ==========
        print("\n📝 1. CREATE — Создание записей")
        print("-" * 50)
        
        # Создание пользователей
        user1 = UserCRUD.create(session, name="Иван Петров", email="ivan@example.com", age=25)
        user2 = UserCRUD.create(session, name="Мария Сидорова", email="maria@example.com", age=30)
        user3 = UserCRUD.create(session, name="Петр Иванов", email="petr@example.com", age=20)
        
        print(f"   Созданы пользователи: {user1.name}, {user2.name}, {user3.name}")
        
        # Массовое создание
        users_data = [
            {"name": "Анна", "email": "anna@example.com", "age": 28},
            {"name": "Дмитрий", "email": "dmitry@example.com", "age": 35}
        ]
        new_users = UserCRUD.bulk_create(session, users_data)
        print(f"   Массовое создание: {len(new_users)} пользователей")
        
        # Создание товаров
        product1 = ProductCRUD.create(session, name="Ноутбук", price=50000, stock=10)
        product2 = ProductCRUD.create(session, name="Мышь", price=1500, stock=50)
        product3 = ProductCRUD.create(session, name="Клавиатура", price=3000, stock=30)
        
        print(f"   Созданы товары: {product1.name}, {product2.name}, {product3.name}")
        
        # ========== READ ==========
        print("\n📖 2. READ — Чтение данных")
        print("-" * 50)
        
        # Получение по ID
        user = UserCRUD.get_by_id(session, user1.id)
        print(f"   get_by_id(1): {user.name}")
        
        # Получение по email
        user = UserCRUD.get_by_email(session, "maria@example.com")
        print(f"   get_by_email('maria@example.com'): {user.name}")
        
        # Получение всех с пагинацией
        users = UserCRUD.get_all(session, skip=0, limit=10)
        print(f"   get_all(): {len(users)} пользователей")
        
        # Активные пользователи
        active_users = UserCRUD.get_active_users(session)
        print(f"   get_active_users(): {len(active_users)}")
        
        # Поиск по имени
        search_results = UserCRUD.search_by_name(session, "Иван")
        print(f"   search_by_name('Иван'): {len(search_results)}")
        
        # Товары в диапазоне цен
        products = ProductCRUD.get_products_by_price_range(session, 1000, 10000)
        print(f"   get_products_by_price_range(1000-10000): {len(products)} товаров")
        
        # Поиск товаров
        search_products = ProductCRUD.search_products(session, "ноут")
        print(f"   search_products('ноут'): {len(search_products)}")
        
        # ========== UPDATE ==========
        print("\n✏️ 3. UPDATE — Обновление данных")
        print("-" * 50)
        
        # Обновление пользователя
        updated = UserCRUD.update(session, user1.id, age=26, name="Иван Сергеевич Петров")
        print(f"   Обновлён пользователь: {updated.name} (возраст: {updated.age})")
        
        # Массовое обновление статуса
        user_ids = [user1.id, user2.id]
        count = UserCRUD.bulk_update_active(session, user_ids, True)
        print(f"   Массовое обновление активности: {count} пользователей")
        
        # Обновление товара
        ProductCRUD.update(session, product1.id, price=45000)
        print(f"   Обновлён товар: {product1.name} (новая цена: {product1.price})")
        
        # Обновление остатка
        ProductCRUD.update_stock(session, product1.id, 2, "reduce")
        print(f"   Обновлён остаток: {product1.name} (остаток: {product1.stock})")
        
        # ========== DELETE ==========
        print("\n🗑️ 4. DELETE — Удаление данных")
        print("-" * 50)
        
        # Удаление пользователя (создадим временного)
        temp_user = UserCRUD.create(session, name="Temp", email="temp@example.com", age=99)
        deleted = UserCRUD.delete(session, temp_user.id)
        print(f"   Удаление пользователя: {'успешно' if deleted else 'неудачно'}")
        
        # ========== СТАТИСТИКА ==========
        print("\n📊 5. СТАТИСТИКА")
        print("-" * 50)
        
        stats = UserCRUD.get_statistics(session)
        print(f"   Пользователи:")
        print(f"      Всего: {stats['total']}")
        print(f"      Активных: {stats['active']}")
        print(f"      Средний возраст: {stats['avg_age']}")
        print(f"      Возраст: {stats['min_age']} - {stats['max_age']}")
        
        product_stats = session.query(
            func.count(Product.id).label('total'),
            func.sum(Product.stock).label('total_stock'),
            func.avg(Product.price).label('avg_price')
        ).first()
        
        print(f"\n   Товары:")
        print(f"      Всего: {product_stats.total}")
        print(f"      Общий остаток: {product_stats.total_stock}")
        print(f"      Средняя цена: {product_stats.avg_price:.2f}")
        
        # ========== ORDER ==========
        print("\n🛒 6. РАБОТА С ЗАКАЗАМИ")
        print("-" * 50)
        
        # Создание заказа
        order = OrderCRUD.create_order(session, user1.id)
        print(f"   Создан заказ #{order.id}")
        
        # Добавление товаров
        item1 = OrderCRUD.add_item(session, order.id, product1.id, 1)
        item2 = OrderCRUD.add_item(session, order.id, product2.id, 2)
        print(f"   Добавлены товары: {product1.name} x1, {product2.name} x2")
        
        # Получение заказа с товарами
        order_data = OrderCRUD.get_order_with_items(session, order.id)
        if order_data:
            print(f"   Заказ #{order_data['id']}: сумма = {order_data['total']}")
            for item in order_data['items']:
                print(f"      - {item['product_name']} x{item['quantity']} = {item['subtotal']}")
        
        # Изменение статуса
        OrderCRUD.update_status(session, order.id, "processing")
        print(f"   Статус заказа изменён на: processing")
        
        # ========== ПАГИНАЦИЯ ==========
        print("\n📄 7. ПАГИНАЦИЯ")
        print("-" * 50)
        
        page_size = 2
        for page in range(1, 4):
            users = UserCRUD.get_all(session, skip=(page-1)*page_size, limit=page_size)
            print(f"   Страница {page}: {len(users)} пользователей")
            for u in users:
                print(f"      - {u.name}")
        
        print("\n" + "=" * 70)
        print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
        print("=" * 70)


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
======================================================================
ДЕМОНСТРАЦИЯ CRUD ОПЕРАЦИЙ ЧЕРЕЗ SQLALCHEMY
======================================================================

📝 1. CREATE — Создание записій
--------------------------------------------------
   Созданы пользователи: Иван Петров, Мария Сидорова, Петр Иванов
   Массовое создание: 2 пользователей
   Созданы товары: Ноутбук, Мышь, Клавиатура

📖 2. READ — Чтение данных
--------------------------------------------------
   get_by_id(1): Иван Петров
   get_by_email('maria@example.com'): Мария Сидорова
   get_all(): 5 пользователей
   get_active_users(): 5
   search_by_name('Иван'): 1
   get_products_by_price_range(1000-10000): 2 товаров
   search_products('ноут'): 1

✏️ 3. UPDATE — Обновление данных
--------------------------------------------------
   Обновлён пользователь: Иван Сергеевич Петров (возраст: 26)
   Массовое обновление активности: 2 пользователей
   Обновлён товар: Ноутбук (новая цена: 45000)
   Обновлён остаток: Ноутбук (остаток: 8)

🗑️ 4. DELETE — Удаление данных
--------------------------------------------------
   Удаление пользователя: успешно

📊 5. СТАТИСТИКА
--------------------------------------------------
   Пользователи:
      Всего: 5
      Активных: 5
      Средний возраст: 27.6
      Возраст: 20 - 35

   Товары:
      Всего: 3
      Общий остаток: 88
      Средняя цена: 16500.00

🛒 6. РАБОТА С ЗАКАЗАМИ
--------------------------------------------------
   Создан заказ #1
   Добавлены товары: Ноутбук x1, Мышь x2
   Заказ #1: сумма = 48000.0
      - Ноутбук x1 = 45000.0
      - Мышь x2 = 3000.0
   Статус заказа изменён на: processing

📄 7. ПАГИНАЦИЯ
--------------------------------------------------
   Страница 1: 2 пользователей
      - Иван Сергеевич Петров
      - Мария Сидорова
   Страница 2: 2 пользователей
      - Петр Иванов
      - Анна
   Страница 3: 1 пользователей
      - Дмитрий

======================================================================
ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА
======================================================================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (CRUD для одной модели)

Варианты 9-17: средний (CRUD + связи)

Варианты 18-25: сложный (CRUD + агрегации + транзакции)

Варианты 1-8 (Базовый уровень)
№	Модель	Поля	CRUD операции
1	User	name, email, age	✅
2	Product	name, price, stock	✅
3	Post	title, content, author	✅
4	Task	title, description, completed	✅
5	Book	title, author, year	✅
6	Movie	title, director, rating	✅
7	Student	name, group, score	✅
8	Employee	name, position, salary	✅
Варианты 9-17 (Средний уровень)
№	Модели	CRUD операции	Связи
9	User, Post	✅ + поиск по автору	User → Post
10	Author, Book	✅ + поиск по автору	Author → Book
11	Category, Product	✅ + фильтрация по категории	Category → Product
12	Customer, Order	✅ + расчёт суммы заказа	Customer → Order
13	Doctor, Appointment	✅ + фильтрация по дате	Doctor → Appointment
14	Playlist, Song	✅ + поиск по плейлисту	Playlist ↔ Song
15	Team, Player	✅ + статистика команды	Team → Player
16	Event, Ticket	✅ + проверка наличия	Event → Ticket
17	Article, Comment	✅ + подсчёт комментариев	Article → Comment
Варианты 18-25 (Сложный уровень)
№	Тема	Особенности
18	Интернет-магазин	User, Product, Order, OrderItem + отчёты
19	Блог	User, Post, Comment, Tag + агрегации
20	Библиотека	User, Book, Loan, Author + штрафы
21	CRM	User, Customer, Deal, Activity + воронка
22	Образование	Student, Course, Teacher, Grade + успеваемость
23	Доставка	User, Order, Courier, Delivery + трекинг
24	Финансы	User, Account, Transaction, Category + баланс
25	Склад	Product, Supplier, Warehouse, Stock + резервирование
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	CRUD не работает или код не запускается
3 (удовлетворительно)	Реализованы базовые CRUD (create, read, update, delete)
4 (хорошо)	+ поиск, фильтрация, пагинация
5 (отлично)	+ массовые операции, агрегации, транзакции
5. Шпаргалка
python
# === БАЗОВЫЙ CRUD ===
session.add(instance)           # CREATE
session.query(Model).get(id)    # READ (один)
session.query(Model).all()      # READ (все)
session.query(Model).filter(...)# READ (фильтр)
instance.attr = value           # UPDATE
session.delete(instance)        # DELETE
session.commit()                # Фиксация
session.rollback()              # Откат

# === ФИЛЬТРАЦИЯ ===
Model.name == "value"           # Равенство
Model.age > 18                  # Больше
Model.age.between(18, 65)       # Диапазон
Model.name.contains("text")     # Содержит
Model.name.in_(['a', 'b'])      # В списке
and_(cond1, cond2)              # И
or_(cond1, cond2)               # ИЛИ

# === АГРЕГАЦИИ ===
func.count(Model.id)            # Количество
func.sum(Model.price)           # Сумма
func.avg(Model.age)             # Среднее
func.min(Model.age)             # Минимум
func.max(Model.age)             # Максимум

# === МАССОВЫЕ ОПЕРАЦИИ ===
session.add_all([obj1, obj2])   # Массовое добавление
session.query(Model).update(...)# Массовое обновление
session.query(Model).delete()   # Массовое удаление
Карточка студента
text
ПЗ 2.29. РЕАЛИЗАЦИЯ CRUD-ОПЕРАЦИЙ ЧЕРЕЗ SQLALCHEMY

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== CRUD ОПЕРАЦИИ ===

