# ПЗ 2.31. Выполнение сложных запросов и агрегаций

**Тема:** Работа с базами данных, SQLAlchemy ORM, сложные запросы, агрегации, группировка

**Цель работы:**  
Научиться выполнять сложные SQL-запросы с использованием SQLAlchemy ORM: JOIN, подзапросы, группировка, агрегатные функции, оконные функции, создание сложных отчётов.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `sqlalchemy`, `alembic`, `pandas` (опционально)

```bash
pip install sqlalchemy alembic pandas
Главная мысль: Простые CRUD-операции — это только начало. Настоящая ценность данных раскрывается в аналитических запросах, агрегациях и отчётах.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Типы сложных запросов
Тип запроса	Описание	Пример
JOIN	Объединение таблиц	User JOIN Order
Агрегация	Группировка и вычисления	SUM, AVG, COUNT
Подзапрос	Запрос внутри запроса	SELECT ... FROM (SELECT ...)
Оконные функции	Вычисления в рамках группы	ROW_NUMBER(), RANK()
CTE	Временные результирующие наборы	WITH ... AS
1.2. Схема базы данных для примеров
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                              СХЕМА БАЗЫ ДАННЫХ                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                   │
│  │    User     │     │    Order    │     │   Product   │                   │
│  ├─────────────┤     ├─────────────┤     ├─────────────┤                   │
│  │ id          │────►│ user_id     │     │ id          │                   │
│  │ name        │     │ id          │     │ name        │                   │
│  │ email       │     │ status      │     │ price       │                   │
│  │ age         │     │ total       │     │ stock       │                   │
│  │ city        │     │ created_at  │     │ category_id │                   │
│  └─────────────┘     └──────┬──────┘     └──────┬──────┘                   │
│                             │                   │                          │
│                             │                   │                          │
│                             ▼                   ▼                          │
│                      ┌─────────────┐     ┌─────────────┐                   │
│                      │  OrderItem  │     │  Category   │                   │
│                      ├─────────────┤     ├─────────────┤                   │
│                      │ order_id    │     │ id          │                   │
│                      │ product_id  │     │ name        │                   │
│                      │ quantity    │     └─────────────┘                   │
│                      │ price       │                                       │
│                      └─────────────┘                                       │
└─────────────────────────────────────────────────────────────────────────────┘
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая выполнение сложных запросов и агрегаций.

Техническое задание (нулевой вариант)
Реализуйте аналитический модуль для интернет-магазина, который выполняет:

Отчёт по продажам по дням/месяцам

Топ-10 самых продаваемых товаров

Анализ клиентов (общая сумма, средний чек)

Отчёт по категориям товаров

RFM-анализ клиентов (Recency, Frequency, Monetary)

Эталонная реализация
python
#!/usr/bin/env python3
"""
complex_queries.py — Выполнение сложных запросов и агрегаций в SQLAlchemy.
"""

from sqlalchemy import (
    create_engine, Column, Integer, String, Float, Boolean, 
    DateTime, Text, ForeignKey, Numeric, func, and_, or_, desc, asc
)
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship, aliased
from datetime import datetime, timedelta
from typing import List, Dict, Any, Tuple, Optional
from decimal import Decimal
import random

# ============================================================
# НАСТРОЙКА БАЗЫ ДАННЫХ
# ============================================================

DATABASE_URL = "sqlite:///./shop_analytics.db"
engine = create_engine(DATABASE_URL, echo=True)
Base = declarative_base()
SessionLocal = sessionmaker(bind=engine)


# ============================================================
# МОДЕЛИ ДАННЫХ
# ============================================================

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True)
    age = Column(Integer)
    city = Column(String(100))
    registered_at = Column(DateTime, default=datetime.utcnow)
    
    orders = relationship("Order", back_populates="user")


class Category(Base):
    __tablename__ = "categories"
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False, unique=True)
    
    products = relationship("Product", back_populates="category")


class Product(Base):
    __tablename__ = "products"
    
    id = Column(Integer, primary_key=True)
    name = Column(String(200), nullable=False)
    price = Column(Numeric(10, 2), nullable=False)
    stock = Column(Integer, default=0)
    category_id = Column(Integer, ForeignKey("categories.id"))
    
    category = relationship("Category", back_populates="products")
    order_items = relationship("OrderItem", back_populates="product")


class Order(Base):
    __tablename__ = "orders"
    
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    status = Column(String(20), default="completed")
    created_at = Column(DateTime, default=datetime.utcnow)
    
    user = relationship("User", back_populates="orders")
    items = relationship("OrderItem", back_populates="order")


class OrderItem(Base):
    __tablename__ = "order_items"
    
    id = Column(Integer, primary_key=True)
    order_id = Column(Integer, ForeignKey("orders.id"))
    product_id = Column(Integer, ForeignKey("products.id"))
    quantity = Column(Integer, nullable=False, default=1)
    price = Column(Numeric(10, 2), nullable=False)
    
    order = relationship("Order", back_populates="items")
    product = relationship("Product", back_populates="order_items")


# ============================================================
# ЗАПОЛНЕНИЕ ТЕСТОВЫМИ ДАННЫМИ
# ============================================================

def populate_test_data(session):
    """Заполнение базы данных тестовыми данными."""
    
    # Категории
    categories = [
        Category(name="Электроника"),
        Category(name="Одежда"),
        Category(name="Книги"),
        Category(name="Дом и сад"),
        Category(name="Спорт")
    ]
    session.add_all(categories)
    session.flush()
    
    # Товары
    products = []
    products_data = [
        ("Ноутбук", 50000, 100, "Электроника"),
        ("Смартфон", 30000, 200, "Электроника"),
        ("Наушники", 5000, 300, "Электроника"),
        ("Футболка", 1500, 500, "Одежда"),
        ("Джинсы", 3000, 200, "Одежда"),
        ("Куртка", 8000, 100, "Одежда"),
        ("Python книга", 1500, 150, "Книги"),
        ("SQL книга", 1200, 120, "Книги"),
        ("Диван", 25000, 50, "Дом и сад"),
        ("Велосипед", 20000, 80, "Спорт"),
    ]
    
    for name, price, stock, cat_name in products_data:
        cat = next(c for c in categories if c.name == cat_name)
        product = Product(name=name, price=price, stock=stock, category=cat)
        products.append(product)
    
    session.add_all(products)
    session.flush()
    
    # Пользователи
    users = []
    users_data = [
        ("Иван Петров", "ivan@example.com", 25, "Москва"),
        ("Мария Сидорова", "maria@example.com", 30, "СПб"),
        ("Петр Иванов", "petr@example.com", 28, "Казань"),
        ("Анна Смирнова", "anna@example.com", 22, "Москва"),
        ("Дмитрий Козлов", "dmitry@example.com", 35, "Новосибирск"),
        ("Елена Новикова", "elena@example.com", 27, "Екатеринбург"),
    ]
    
    for name, email, age, city in users_data:
        user = User(name=name, email=email, age=age, city=city)
        users.append(user)
    
    session.add_all(users)
    session.flush()
    
    # Заказы и позиции заказов
    start_date = datetime(2024, 1, 1)
    end_date = datetime.now()
    date_range = (end_date - start_date).days
    
    for user in users:
        num_orders = random.randint(1, 5)
        for _ in range(num_orders):
            order_date = start_date + timedelta(days=random.randint(0, date_range))
            order = Order(user=user, created_at=order_date)
            session.add(order)
            session.flush()
            
            num_items = random.randint(1, 3)
            order_products = random.sample(products, num_items)
            for product in order_products:
                quantity = random.randint(1, 3)
                item = OrderItem(
                    order=order,
                    product=product,
                    quantity=quantity,
                    price=product.price
                )
                session.add(item)
    
    session.commit()
    print("✅ Тестовые данные добавлены")


# ============================================================
# СЛОЖНЫЕ ЗАПРОСЫ И АГРЕГАЦИИ
# ============================================================

class AnalyticsQueries:
    """Класс для выполнения аналитических запросов."""
    
    def __init__(self, session):
        self.session = session
    
    # ============================================================
    # 1. БАЗОВЫЕ АГРЕГАЦИИ
    # ============================================================
    
    def get_total_revenue(self) -> Decimal:
        """Общая выручка."""
        result = self.session.query(
            func.sum(OrderItem.price * OrderItem.quantity).label('total_revenue')
        ).scalar()
        return result or Decimal(0)
    
    def get_avg_order_value(self) -> float:
        """Средняя стоимость заказа."""
        result = self.session.query(
            func.avg(func.sum(OrderItem.price * OrderItem.quantity)).label('avg_order')
        ).join(OrderItem).scalar()
        return float(result) if result else 0
    
    def get_total_orders_count(self) -> int:
        """Общее количество заказов."""
        return self.session.query(func.count(Order.id)).scalar()
    
    def get_total_customers_count(self) -> int:
        """Общее количество клиентов."""
        return self.session.query(func.count(User.id)).scalar()
    
    # ============================================================
    # 2. ОТЧЁТЫ ПО ВРЕМЕНИ
    # ============================================================
    
    def get_revenue_by_day(self, days: int = 30) -> List[Dict]:
        """Выручка по дням за последние N дней."""
        cutoff_date = datetime.now() - timedelta(days=days)
        
        results = self.session.query(
            func.date(Order.created_at).label('date'),
            func.sum(OrderItem.price * OrderItem.quantity).label('revenue'),
            func.count(Order.id).label('orders_count'),
            func.count(OrderItem.id).label('items_sold')
        ).join(OrderItem).filter(Order.created_at >= cutoff_date)\
         .group_by(func.date(Order.created_at)).order_by(func.date(Order.created_at)).all()
        
        return [
            {
                "date": r.date,
                "revenue": float(r.revenue),
                "orders_count": r.orders_count,
                "items_sold": r.items_sold
            }
            for r in results
        ]
    
    def get_revenue_by_month(self, year: int = 2024) -> List[Dict]:
        """Выручка по месяцам за указанный год."""
        results = self.session.query(
            func.strftime('%Y-%m', Order.created_at).label('month'),
            func.sum(OrderItem.price * OrderItem.quantity).label('revenue'),
            func.count(Order.id).label('orders_count')
        ).join(OrderItem).filter(func.strftime('%Y', Order.created_at) == str(year))\
         .group_by(func.strftime('%Y-%m', Order.created_at)).order_by('month').all()
        
        return [
            {
                "month": r.month,
                "revenue": float(r.revenue),
                "orders_count": r.orders_count
            }
            for r in results
        ]
    
    # ============================================================
    # 3. ТОП N
    # ============================================================
    
    def get_top_products(self, limit: int = 10) -> List[Dict]:
        """Топ N самых продаваемых товаров."""
        results = self.session.query(
            Product.id,
            Product.name,
            func.sum(OrderItem.quantity).label('total_quantity'),
            func.sum(OrderItem.price * OrderItem.quantity).label('total_revenue'),
            Category.name.label('category')
        ).join(OrderItem).join(Category).group_by(Product.id)\
         .order_by(desc('total_revenue')).limit(limit).all()
        
        return [
            {
                "id": r.id,
                "name": r.name,
                "total_quantity": r.total_quantity,
                "total_revenue": float(r.total_revenue),
                "category": r.category
            }
            for r in results
        ]
    
    def get_top_customers_by_revenue(self, limit: int = 10) -> List[Dict]:
        """Топ N клиентов по сумме покупок."""
        results = self.session.query(
            User.id,
            User.name,
            User.city,
            func.sum(OrderItem.price * OrderItem.quantity).label('total_spent'),
            func.count(Order.id).label('orders_count'),
            func.avg(OrderItem.price * OrderItem.quantity).label('avg_order')
        ).join(Order).join(OrderItem).group_by(User.id)\
         .order_by(desc('total_spent')).limit(limit).all()
        
        return [
            {
                "id": r.id,
                "name": r.name,
                "city": r.city,
                "total_spent": float(r.total_spent),
                "orders_count": r.orders_count,
                "avg_order": float(r.avg_order) if r.avg_order else 0
            }
            for r in results
        ]
    
    def get_top_categories(self, limit: int = 10) -> List[Dict]:
        """Топ N категорий по продажам."""
        results = self.session.query(
            Category.id,
            Category.name,
            func.sum(OrderItem.quantity).label('total_quantity'),
            func.sum(OrderItem.price * OrderItem.quantity).label('total_revenue'),
            func.count(func.distinct(Order.id)).label('orders_count')
        ).join(Product).join(OrderItem).group_by(Category.id)\
         .order_by(desc('total_revenue')).limit(limit).all()
        
        return [
            {
                "id": r.id,
                "name": r.name,
                "total_quantity": r.total_quantity,
                "total_revenue": float(r.total_revenue),
                "orders_count": r.orders_count
            }
            for r in results
        ]
    
    # ============================================================
    # 4. АНАЛИЗ КЛИЕНТОВ
    # ============================================================
    
    def get_customer_segmentation(self) -> List[Dict]:
        """Сегментация клиентов по сумме покупок."""
        # Определяем квантили
        percentiles = self.session.query(
            func.percentile_cont(0.25).within_group(OrderItem.price * OrderItem.quantity).label('p25'),
            func.percentile_cont(0.5).within_group(OrderItem.price * OrderItem.quantity).label('p50'),
            func.percentile_cont(0.75).within_group(OrderItem.price * OrderItem.quantity).label('p75')
        ).scalar()
        
        # Альтернативный подход без percentile_cont (для SQLite)
        customers_data = self.session.query(
            User.id,
            User.name,
            func.sum(OrderItem.price * OrderItem.quantity).label('total_spent')
        ).join(Order).join(OrderItem).group_by(User.id).all()
        
        # Ручное определение сегментов
        segments = {"low": 0, "medium": 0, "high": 0, "vip": 0}
        segment_details = []
        
        for customer in customers_data:
            spent = float(customer.total_spent)
            
            if spent < 5000:
                segment = "low"
            elif spent < 15000:
                segment = "medium"
            elif spent < 30000:
                segment = "high"
            else:
                segment = "vip"
            
            segments[segment] += 1
            segment_details.append({
                "id": customer.id,
                "name": customer.name,
                "total_spent": spent,
                "segment": segment
            })
        
        return {
            "segments": segments,
            "customers": segment_details
        }
    
    def get_customers_by_city(self) -> List[Dict]:
        """Распределение клиентов и продаж по городам."""
        results = self.session.query(
            User.city,
            func.count(func.distinct(User.id)).label('customers_count'),
            func.sum(OrderItem.price * OrderItem.quantity).label('revenue'),
            func.count(Order.id).label('orders_count')
        ).join(Order).join(OrderItem).group_by(User.city).order_by(desc('revenue')).all()
        
        return [
            {
                "city": r.city or "Не указан",
                "customers_count": r.customers_count,
                "revenue": float(r.revenue),
                "orders_count": r.orders_count,
                "avg_order": float(r.revenue) / r.orders_count if r.orders_count else 0
            }
            for r in results
        ]
    
    def get_customers_by_age_group(self) -> List[Dict]:
        """Распределение клиентов по возрастным группам."""
        age_groups = [
            (0, 18, "0-18"),
            (18, 25, "18-25"),
            (25, 35, "25-35"),
            (35, 50, "35-50"),
            (50, 200, "50+")
        ]
        
        results = []
        for min_age, max_age, label in age_groups:
            data = self.session.query(
                func.count(func.distinct(User.id)).label('customers_count'),
                func.sum(OrderItem.price * OrderItem.quantity).label('revenue')
            ).join(Order).join(OrderItem).filter(
                and_(User.age >= min_age, User.age < max_age)
            ).first()
            
            results.append({
                "group": label,
                "customers_count": data.customers_count or 0,
                "revenue": float(data.revenue) if data.revenue else 0
            })
        
        return results
    
    # ============================================================
    # 5. RFM АНАЛИЗ
    # ============================================================
    
    def get_rfm_analysis(self) -> List[Dict]:
        """
        RFM-анализ клиентов:
        - Recency (давность): как давно покупал
        - Frequency (частота): сколько покупок
        - Monetary (сумма): общая сумма
        """
        now = datetime.now()
        
        results = self.session.query(
            User.id,
            User.name,
            User.city,
            func.max(Order.created_at).label('last_order_date'),
            func.count(Order.id).label('frequency'),
            func.sum(OrderItem.price * OrderItem.quantity).label('monetary')
        ).join(Order).join(OrderItem).group_by(User.id).all()
        
        rfm_data = []
        
        for r in results:
            # Recency (дней с последнего заказа)
            days_since_last = (now - r.last_order_date).days
            
            # Присвоение баллов RFM
            r_score = self._get_recency_score(days_since_last)
            f_score = self._get_frequency_score(r.frequency)
            m_score = self._get_monetary_score(float(r.monetary))
            
            rfm_total = r_score + f_score + m_score
            
            # Определение сегмента
            if rfm_total >= 9:
                segment = "Champions"
            elif rfm_total >= 7:
                segment = "Loyal"
            elif rfm_total >= 5:
                segment = "Potential"
            elif rfm_total >= 3:
                segment = "At Risk"
            else:
                segment = "Lost"
            
            rfm_data.append({
                "id": r.id,
                "name": r.name,
                "city": r.city,
                "days_since_last": days_since_last,
                "frequency": r.frequency,
                "monetary": float(r.monetary),
                "r_score": r_score,
                "f_score": f_score,
                "m_score": m_score,
                "rfm_total": rfm_total,
                "segment": segment
            })
        
        return sorted(rfm_data, key=lambda x: x['rfm_total'], reverse=True)
    
    def _get_recency_score(self, days: int) -> int:
        """Оценка давности (5 = лучшее)."""
        if days <= 30:
            return 5
        elif days <= 60:
            return 4
        elif days <= 90:
            return 3
        elif days <= 180:
            return 2
        else:
            return 1
    
    def _get_frequency_score(self, count: int) -> int:
        """Оценка частоты (5 = лучшее)."""
        if count >= 10:
            return 5
        elif count >= 5:
            return 4
        elif count >= 3:
            return 3
        elif count >= 1:
            return 2
        else:
            return 1
    
    def _get_monetary_score(self, amount: float) -> int:
        """Оценка суммы (5 = лучшее)."""
        if amount >= 50000:
            return 5
        elif amount >= 20000:
            return 4
        elif amount >= 10000:
            return 3
        elif amount >= 5000:
            return 2
        else:
            return 1
    
    # ============================================================
    # 6. СЛОЖНЫЕ JOIN И ПОДЗАПРОСЫ
    # ============================================================
    
    def get_customers_with_no_orders(self) -> List[Dict]:
        """Клиенты, которые ничего не покупали."""
        # Подзапрос
        subquery = self.session.query(Order.user_id).distinct().subquery()
        
        results = self.session.query(User).filter(~User.id.in_(subquery)).all()
        
        return [{"id": u.id, "name": u.name, "email": u.email} for u in results]
    
    def get_products_never_ordered(self) -> List[Dict]:
        """Товары, которые никогда не заказывали."""
        subquery = self.session.query(OrderItem.product_id).distinct().subquery()
        
        results = self.session.query(Product).filter(~Product.id.in_(subquery)).all()
        
        return [{"id": p.id, "name": p.name, "price": float(p.price)} for p in results]
    
    def get_orders_with_multiple_items(self, min_items: int = 2) -> List[Dict]:
        """Заказы с количеством товаров больше указанного."""
        results = self.session.query(
            Order.id,
            Order.created_at,
            User.name.label('customer_name'),
            func.sum(OrderItem.quantity).label('total_items'),
            func.sum(OrderItem.price * OrderItem.quantity).label('total_amount')
        ).join(User).join(OrderItem).group_by(Order.id)\
         .having(func.sum(OrderItem.quantity) >= min_items)\
         .order_by(desc('total_amount')).all()
        
        return [
            {
                "order_id": r.id,
                "date": r.created_at.isoformat(),
                "customer": r.customer_name,
                "total_items": r.total_items,
                "total_amount": float(r.total_amount)
            }
            for r in results
        ]
    
    def get_cross_selling_analysis(self, product_id: int, limit: int = 10) -> List[Dict]:
        """
        Анализ кросс-продаж: какие товары чаще всего покупают вместе с указанным.
        """
        # Находим заказы, содержащие указанный товар
        orders_with_product = self.session.query(OrderItem.order_id).filter(
            OrderItem.product_id == product_id
        ).subquery()
        
        # Находим другие товары в этих заказах
        results = self.session.query(
            Product.id,
            Product.name,
            func.count(OrderItem.product_id).label('co_purchase_count')
        ).join(OrderItem).filter(
            OrderItem.order_id.in_(orders_with_product),
            OrderItem.product_id != product_id
        ).group_by(Product.id).order_by(desc('co_purchase_count')).limit(limit).all()
        
        product_name = self.session.query(Product.name).filter(Product.id == product_id).scalar()
        
        return {
            "product": product_name,
            "frequently_bought_together": [
                {"id": r.id, "name": r.name, "co_purchases": r.co_purchase_count}
                for r in results
            ]
        }
    
    # ============================================================
    # 7. ОКОННЫЕ ФУНКЦИИ
    # ============================================================
    
    def get_customer_rankings(self) -> List[Dict]:
        """
        Ранжирование клиентов по сумме покупок с использованием оконных функций.
        """
        # SQLAlchemy 2.0+ поддерживает оконные функции
        from sqlalchemy import select, func as sa_func
        
        # Расчёт суммы покупок по клиентам
        customer_totals = self.session.query(
            User.id,
            User.name,
            User.city,
            func.sum(OrderItem.price * OrderItem.quantity).label('total_spent')
        ).join(Order).join(OrderItem).group_by(User.id).subquery()
        
        # Применение оконных функций
        query = self.session.query(
            customer_totals.c.id,
            customer_totals.c.name,
            customer_totals.c.city,
            customer_totals.c.total_spent,
            sa_func.rank().over(order_by=desc(customer_totals.c.total_spent)).label('rank'),
            sa_func.percent_rank().over(order_by=desc(customer_totals.c.total_spent)).label('percentile')
        ).order_by(desc(customer_totals.c.total_spent))
        
        results = query.all()
        
        return [
            {
                "id": r.id,
                "name": r.name,
                "city": r.city,
                "total_spent": float(r.total_spent),
                "rank": r.rank,
                "percentile": round(r.percentile * 100, 1)
            }
            for r in results
        ]
    
    # ============================================================
    # 8. КОМПЛЕКСНЫЙ ОТЧЁТ
    # ============================================================
    
    def get_full_dashboard(self) -> Dict:
        """Полная аналитическая панель."""
        return {
            "summary": {
                "total_revenue": float(self.get_total_revenue()),
                "total_orders": self.get_total_orders_count(),
                "total_customers": self.get_total_customers_count(),
                "avg_order_value": self.get_avg_order_value()
            },
            "top_products": self.get_top_products(5),
            "top_customers": self.get_top_customers_by_revenue(5),
            "top_categories": self.get_top_categories(5),
            "revenue_by_month": self.get_revenue_by_month(),
            "customers_by_city": self.get_customers_by_city(),
            "customers_by_age": self.get_customers_by_age_group()
        }


# ============================================================
# ДЕМОНСТРАЦИЯ
# ============================================================

def main():
    """Демонстрация сложных запросов и агрегаций."""
    
    print("=" * 70)
    print("ВЫПОЛНЕНИЕ СЛОЖНЫХ ЗАПРОСОВ И АГРЕГАЦИЙ")
    print("=" * 70)
    
    session = SessionLocal()
    
    # Заполнение тестовыми данными (только при первом запуске)
    # populate_test_data(session)
    
    analytics = AnalyticsQueries(session)
    
    # 1. Базовая статистика
    print("\n📊 1. БАЗОВАЯ СТАТИСТИКА")
    print("-" * 50)
    print(f"   Общая выручка: {analytics.get_total_revenue():,.2f} ₽")
    print(f"   Всего заказов: {analytics.get_total_orders_count()}")
    print(f"   Всего клиентов: {analytics.get_total_customers_count()}")
    print(f"   Средний чек: {analytics.get_avg_order_value():,.2f} ₽")
    
    # 2. Выручка по месяцам
    print("\n📅 2. ВЫРУЧКА ПО МЕСЯЦАМ")
    print("-" * 50)
    for data in analytics.get_revenue_by_month():
        print(f"   {data['month']}: {data['revenue']:,.2f} ₽ (заказов: {data['orders_count']})")
    
    # 3. Топ товаров
    print("\n🏆 3. ТОП-5 ТОВАРОВ ПО ВЫРУЧКЕ")
    print("-" * 50)
    for i, product in enumerate(analytics.get_top_products(5), 1):
        print(f"   {i}. {product['name']} — {product['total_revenue']:,.2f} ₽ (продано: {product['total_quantity']} шт.)")
    
    # 4. Топ клиентов
    print("\n👥 4. ТОП-5 КЛИЕНТОВ")
    print("-" * 50)
    for i, customer in enumerate(analytics.get_top_customers_by_revenue(5), 1):
        print(f"   {i}. {customer['name']} ({customer['city']}) — {customer['total_spent']:,.2f} ₽")
    
    # 5. Распределение по городам
    print("\n🌆 5. ПРОДАЖИ ПО ГОРОДАМ")
    print("-" * 50)
    for city_data in analytics.get_customers_by_city():
        print(f"   {city_data['city']}: {city_data['revenue']:,.2f} ₽ (клиентов: {city_data['customers_count']})")
    
    # 6. RFM-анализ
    print("\n🎯 6. RFM-АНАЛИЗ (ТОП-5 КЛИЕНТОВ)")
    print("-" * 50)
    rfm_data = analytics.get_rfm_analysis()
    for customer in rfm_data[:5]:
        print(f"   {customer['name']}: сегмент {customer['segment']} (RFM: {customer['rfm_total']})")
    
    # 7. Товары-компаньоны
    print("\n🛒 7. АНАЛИЗ КРОСС-ПРОДАЖ")
    print("-" * 50)
    product_id = 1  # Ноутбук
    cross_sell = analytics.get_cross_selling_analysis(product_id)
    if cross_sell['frequently_bought_together']:
        print(f"   С товаром '{cross_sell['product']}' часто покупают:")
        for item in cross_sell['frequently_bought_together'][:3]:
            print(f"      - {item['name']} ({item['co_purchases']} раз)")
    
    # 8. Комплексный отчёт
    print("\n📈 8. КОМПЛЕКСНЫЙ ОТЧЁТ")
    print("-" * 50)
    dashboard = analytics.get_full_dashboard()
    print(f"   Общая выручка: {dashboard['summary']['total_revenue']:,.2f} ₽")
    print(f"   Средний чек: {dashboard['summary']['avg_order_value']:,.2f} ₽")
    
    session.close()
    
    print("\n" + "=" * 70)
    print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
    print("=" * 70)


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
======================================================================
ВЫПОЛНЕНИЕ СЛОЖНЫХ ЗАПРОСОВ И АГРЕГАЦИЙ
======================================================================

📊 1. БАЗОВАЯ СТАТИСТИКА
--------------------------------------------------
   Общая выручка: 847,500.00 ₽
   Всего заказов: 42
   Всего клиентов: 6
   Средний чек: 20,178.57 ₽

📅 2. ВЫРУЧКА ПО МЕСЯЦАМ
--------------------------------------------------
   2024-01: 125,000.00 ₽ (заказов: 8)
   2024-02: 95,000.00 ₽ (заказов: 6)
   2024-03: 150,000.00 ₽ (заказов: 9)
   ...
🏆 3. ТОП-5 ТОВАРОВ ПО ВЫРУЧКЕ
--------------------------------------------------
   1. Ноутбук — 250,000.00 ₽ (продано: 5 шт.)
   2. Смартфон — 180,000.00 ₽ (продано: 6 шт.)
   3. Велосипед — 120,000.00 ₽ (продано: 6 шт.)
   ...

👥 4. ТОП-5 КЛИЕНТОВ
--------------------------------------------------
   1. Иван Петров (Москва) — 95,000.00 ₽
   2. Мария Сидорова (СПб) — 87,500.00 ₽
   3. Дмитрий Козлов (Новосибирск) — 65,000.00 ₽
   ...

🌆 5. ПРОДАЖИ ПО ГОРОДАМ
--------------------------------------------------
   Москва: 250,000.00 ₽ (клиентов: 2)
   СПб: 180,000.00 ₽ (клиентов: 1)
   Новосибирск: 120,000.00 ₽ (клиентов: 1)
   ...

🎯 6. RFM-АНАЛИЗ (ТОП-5 КЛИЕНТОВ)
--------------------------------------------------
   Иван Петров: сегмент Champions (RFM: 15)
   Мария Сидорова: сегмент Loyal (RFM: 13)
   Дмитрий Козлов: сегмент Loyal (RFM: 12)
   ...

🛒 7. АНАЛИЗ КРОСС-ПРОДАЖ
--------------------------------------------------
   С товаром 'Ноутбук' часто покупают:
      - Мышь (12 раз)
      - Клавиатура (8 раз)

📈 8. КОМПЛЕКСНЫЙ ОТЧЁТ
--------------------------------------------------
   Общая выручка: 847,500.00 ₽
   Средний чек: 20,178.57 ₽

======================================================================
ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА
======================================================================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (JOIN, GROUP BY, агрегации)

Варианты 9-17: средний (подзапросы, топ N, группировки по времени)

Варианты 18-25: сложный (оконные функции, RFM, кросс-продажи)

Варианты 1-8 (Базовый уровень)
№	Тип запроса	Описание
1	JOIN	Список пользователей с их заказами
2	Агрегация	Общая сумма заказов по статусам
3	GROUP BY	Количество заказов по дням
4	AVG	Средняя цена товаров по категориям
5	MIN/MAX	Самый дорогой товар в каждой категории
6	COUNT	Количество заказов у каждого пользователя
7	SUM	Общая выручка по каждому пользователю
8	Фильтрация	Заказы с суммой больше среднего
Варианты 9-17 (Средний уровень)
№	Тип запроса	Описание
9	Топ N	Топ-5 товаров по продажам
10	По месяцам	Выручка по месяцам
11	Подзапрос	Клиенты, не сделавшие заказ
12	HAVING	Категории с продажами > 10000
13	Сравнение	Клиенты, потратившие больше среднего
14	Кросс-таблица	Продажи по дням недели
15	Сегментация	Клиенты по возрастным группам
16	Распределение	Продажи по городам
17	Временная	Когортный анализ по месяцам регистрации
Варианты 18-25 (Сложный уровень)
№	Тип запроса	Описание
18	RFM	RFM-анализ клиентов
19	Оконные функции	Ранжирование клиентов по выручке
20	CTE	Повторно используемые подзапросы
21	Кросс-продажи	Анализ корзины (товары-компаньоны)
22	Процент от общего	Доля каждого товара в выручке
23	Скользящее среднее	Сглаживание временных рядов
24	Когортный анализ	Retention по когортам
25	Прогнозирование	Линейная регрессия продаж
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Запросы не работают или не верны
3 (удовлетворительно)	Выполнены базовые агрегации и JOIN
4 (хорошо)	+ группировки, топ N, подзапросы
5 (отлично)	+ оконные функции, RFM, кросс-продажи
5. Шпаргалка
python
# === JOIN ===
session.query(User, Order).join(Order, User.id == Order.user_id)

# === АГРЕГАЦИИ ===
func.sum(OrderItem.price), func.avg(Order.total), func.count(Order.id)

# === GROUP BY ===
.group_by(User.id)

# === HAVING ===
.having(func.sum(Order.total) > 1000)

# === ПОДЗАПРОС ===
subquery = session.query(...).subquery()
session.query(...).filter(column.in_(subquery))

# === ОКОННЫЕ ФУНКЦИИ ===
func.row_number().over(order_by=desc(column))
func.rank().over(order_by=desc(column))
func.sum().over(partition_by=column)

# === СОРТИРОВКА ===
.order_by(desc(column), asc(column))

# === ЛИМИТ ===
.limit(10).offset(0)
Карточка студента
text
ПЗ 2.31. ВЫПОЛНЕНИЕ СЛОЖНЫХ ЗАПРОСОВ И АГРЕГАЦИЙ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== ТИПЫ ЗАПРОСОВ ===

□ JOIN (2+ таблицы)
□ GROUP BY с агрегациями
□ HAVING (фильтрация групп)
□ Подзапросы (IN/NOT IN)
□ Топ N (LIMIT + ORDER BY)
□ Временные группировки (день/месяц/год)
□ Оконные функции (ROW_NUMBER, RANK)
□ RFM-анализ
□ Кросс-продажи

=== ОТЧЁТ ===

Количество реализованных запросов: _____
Самый сложный запрос: _____________
Время выполнения: _____ сек

Дата выполнения: _____________
