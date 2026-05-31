# Тема 2.40. Интеграция всех компонентов: API + БД + ETL

**Цель лекции:**  
Изучить интеграцию всех ключевых компонентов современного приложения: REST API для взаимодействия с пользователями, базы данных для хранения и ETL-процессов для обработки данных. Создать полноценную систему, объединяющую эти компоненты.

> Главная мысль: **API — лицо приложения, БД — память, ETL — мозг. Вместе они создают полноценную, полезную систему.**

---

## Содержание

1. [Архитектура интегрированной системы](#1-архитектура-интегрированной-системы)
2. [Компоненты системы](#2-компоненты-системы)
3. [Интеграция API и БД](#3-интеграция-api-и-бд)
4. [Интеграция ETL-процесса](#4-интеграция-etl-процесса)
5. [Полная интеграция](#5-полная-интеграция)
6. [Практические примеры](#6-практические-примеры)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Архитектура интегрированной системы

### 1.1. Общая архитектура
┌─────────────────────────────────────────────────────────────────────────────┐
│ АРХИТЕКТУРА ИНТЕГРИРОВАННОЙ СИСТЕМЫ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ КЛИЕНТЫ │ │
│ │ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │ │
│ │ │ Web App │ │ Mobile │ │ CLI │ │ Other │ │ │
│ │ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ │ │
│ │ └────────────┼────────────┼────────────┘ │ │
│ │ │ │ │ │
│ │ ▼ ▼ │ │
│ │ ┌─────────────────────────┐ │ │
│ │ │ REST API │ │ │
│ │ │ (FastAPI) │ │ │
│ │ └───────────┬─────────────┘ │ │
│ └──────────────────────────┼─────────────────────────────────────────┘ │
│ │ │
│ ┌──────────────────────────┼─────────────────────────────────────────┐ │
│ │ │ │ │
│ │ ┌───────────────────────┼───────────────────────────────────────┐ │ │
│ │ │ ▼ │ │ │
│ │ │ ┌─────────────────────────────────────────────────────────┐ │ │ │
│ │ │ │ БАЗА ДАННЫХ │ │ │ │
│ │ │ │ (PostgreSQL/SQLite) │ │ │ │
│ │ │ │ │ │ │ │
│ │ │ │ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │ │ │ │
│ │ │ │ │ Users │ │ Orders │ │Products │ │Analytics│ │ │ │ │
│ │ │ │ └─────────┘ └─────────┘ └─────────┘ └─────────┘ │ │ │ │
│ │ │ └─────────────────────────────────────────────────────────┘ │ │ │
│ │ │ │ │ │
│ │ │ ┌─────────────────────────────────────────────────────────┐ │ │ │
│ │ │ │ ETL ПРОЦЕСС │ │ │ │
│ │ │ │ │ │ │ │
│ │ │ │ ┌──────────┐ ┌──────────┐ ┌──────────┐ │ │ │ │
│ │ │ │ │ Extract │───►│Transform │───►│ Load │ │ │ │ │
│ │ │ │ │ (CSV/API)│ │(Аналитика)│ │ (БД) │ │ │ │ │
│ │ │ │ └──────────┘ └──────────┘ └──────────┘ │ │ │ │
│ │ │ └─────────────────────────────────────────────────────────┘ │ │ │
│ │ └───────────────────────────────────────────────────────────────┘ │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Потоки данных

| Направление | Описание | Технологии |
|-------------|----------|------------|
| **API → БД** | Создание/обновление данных через API | FastAPI + SQLAlchemy |
| **БД → API** | Получение данных через API | FastAPI + SQLAlchemy |
| **ETL → БД** | Загрузка обработанных данных | Python + SQLAlchemy |
| **БД → ETL** | Извлечение данных для обработки | SQLAlchemy + Pandas |

### 1.3. Компоненты системы

| Компонент | Ответственность | Технологии |
|-----------|-----------------|------------|
| **REST API** | Взаимодействие с клиентами | FastAPI, Pydantic |
| **База данных** | Хранение данных | SQLite/PostgreSQL, SQLAlchemy |
| **ETL** | Обработка и агрегация данных | Pandas, SQLAlchemy |
| **Конфигурация** | Настройка окружения | python-dotenv |
| **Логирование** | Отслеживание работы | logging |
| **Валидация** | Проверка данных | Pydantic |

---

## 2. Компоненты системы

### 2.1. Модели данных (SQLAlchemy)

```python
# models.py
from sqlalchemy import Column, Integer, String, Float, DateTime, ForeignKey, Boolean, Numeric
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship
from datetime import datetime

Base = declarative_base()


class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    orders = relationship("Order", back_populates="user")


class Product(Base):
    __tablename__ = 'products'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(200), nullable=False)
    price = Column(Numeric(10, 2), nullable=False)
    stock = Column(Integer, default=0)
    category = Column(String(50))
    created_at = Column(DateTime, default=datetime.utcnow)
    
    order_items = relationship("OrderItem", back_populates="product")


class Order(Base):
    __tablename__ = 'orders'
    
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('users.id'))
    status = Column(String(20), default='pending')
    total = Column(Numeric(10, 2), default=0)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    user = relationship("User", back_populates="orders")
    items = relationship("OrderItem", back_populates="order")


class OrderItem(Base):
    __tablename__ = 'order_items'
    
    id = Column(Integer, primary_key=True)
    order_id = Column(Integer, ForeignKey('orders.id'))
    product_id = Column(Integer, ForeignKey('products.id'))
    quantity = Column(Integer, default=1)
    price = Column(Numeric(10, 2), nullable=False)
    
    order = relationship("Order", back_populates="items")
    product = relationship("Product", back_populates="order_items")


class SalesAnalytics(Base):
    __tablename__ = 'sales_analytics'
    
    id = Column(Integer, primary_key=True)
    date = Column(DateTime, nullable=False)
    total_sales = Column(Numeric(10, 2), default=0)
    order_count = Column(Integer, default=0)
    avg_order_value = Column(Numeric(10, 2), default=0)
    top_product_id = Column(Integer, ForeignKey('products.id'))
    created_at = Column(DateTime, default=datetime.utcnow)
2.2. Pydantic схемы для API
python
# schemas.py
from pydantic import BaseModel, Field, EmailStr
from typing import Optional, List
from datetime import datetime


class UserCreate(BaseModel):
    name: str = Field(..., min_length=2, max_length=100)
    email: EmailStr


class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    created_at: datetime
    
    class Config:
        orm_mode = True


class ProductCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=200)
    price: float = Field(..., gt=0)
    stock: int = Field(0, ge=0)
    category: Optional[str] = None


class ProductResponse(ProductCreate):
    id: int
    created_at: datetime
    
    class Config:
        orm_mode = True


class OrderItemCreate(BaseModel):
    product_id: int
    quantity: int = Field(1, ge=1)


class OrderCreate(BaseModel):
    items: List[OrderItemCreate]


class OrderItemResponse(BaseModel):
    id: int
    product_id: int
    product_name: str
    quantity: int
    price: float
    subtotal: float


class OrderResponse(BaseModel):
    id: int
    user_id: int
    status: str
    total: float
    items: List[OrderItemResponse]
    created_at: datetime
    
    class Config:
        orm_mode = True


class SalesAnalyticsResponse(BaseModel):
    date: datetime
    total_sales: float
    order_count: int
    avg_order_value: float
    top_product: Optional[str]
3. Интеграция API и БД
3.1. Основное API приложение
python
# main.py
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy.orm import Session
from typing import List

from database import get_db, engine, Base
from models import User, Product, Order, OrderItem
from schemas import (
    UserCreate, UserResponse,
    ProductCreate, ProductResponse,
    OrderCreate, OrderResponse,
    SalesAnalyticsResponse
)
from services import OrderService, AnalyticsService
import logging

# Настройка логирования
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Создание таблиц
Base.metadata.create_all(bind=engine)

# FastAPI приложение
app = FastAPI(
    title="E-Commerce API",
    description="Интегрированное API с БД и ETL",
    version="1.0.0"
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)


# ============================================================
# USER ENDPOINTS
# ============================================================

@app.post("/users", response_model=UserResponse, status_code=201)
def create_user(user: UserCreate, db: Session = Depends(get_db)):
    """Создание нового пользователя."""
    db_user = db.query(User).filter(User.email == user.email).first()
    if db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    
    db_user = User(name=user.name, email=user.email)
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    
    logger.info(f"User created: {db_user.id}")
    return db_user


@app.get("/users", response_model=List[UserResponse])
def get_users(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    """Получение списка пользователей."""
    users = db.query(User).offset(skip).limit(limit).all()
    return users


@app.get("/users/{user_id}", response_model=UserResponse)
def get_user(user_id: int, db: Session = Depends(get_db)):
    """Получение пользователя по ID."""
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user


# ============================================================
# PRODUCT ENDPOINTS
# ============================================================

@app.post("/products", response_model=ProductResponse, status_code=201)
def create_product(product: ProductCreate, db: Session = Depends(get_db)):
    """Создание нового товара."""
    db_product = Product(**product.dict())
    db.add(db_product)
    db.commit()
    db.refresh(db_product)
    
    logger.info(f"Product created: {db_product.id}")
    return db_product


@app.get("/products", response_model=List[ProductResponse])
def get_products(
    skip: int = 0,
    limit: int = 100,
    category: str = None,
    min_price: float = None,
    db: Session = Depends(get_db)
):
    """Получение списка товаров с фильтрацией."""
    query = db.query(Product)
    
    if category:
        query = query.filter(Product.category == category)
    if min_price:
        query = query.filter(Product.price >= min_price)
    
    products = query.offset(skip).limit(limit).all()
    return products


@app.get("/products/{product_id}", response_model=ProductResponse)
def get_product(product_id: int, db: Session = Depends(get_db)):
    """Получение товара по ID."""
    product = db.query(Product).filter(Product.id == product_id).first()
    if not product:
        raise HTTPException(status_code=404, detail="Product not found")
    return product


# ============================================================
# ORDER ENDPOINTS
# ============================================================

@app.post("/orders/{user_id}", response_model=OrderResponse, status_code=201)
def create_order(
    user_id: int,
    order: OrderCreate,
    db: Session = Depends(get_db)
):
    """Создание заказа для пользователя."""
    # Проверка существования пользователя
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    # Создание заказа
    db_order = Order(user_id=user_id)
    db.add(db_order)
    db.flush()
    
    total = 0
    for item in order.items:
        product = db.query(Product).filter(Product.id == item.product_id).first()
        if not product:
            raise HTTPException(status_code=404, detail=f"Product {item.product_id} not found")
        
        if product.stock < item.quantity:
            raise HTTPException(status_code=400, detail=f"Insufficient stock for {product.name}")
        
        # Уменьшаем остаток
        product.stock -= item.quantity
        
        # Создаём элемент заказа
        order_item = OrderItem(
            order_id=db_order.id,
            product_id=item.product_id,
            quantity=item.quantity,
            price=product.price
        )
        db.add(order_item)
        total += product.price * item.quantity
    
    db_order.total = total
    db.commit()
    db.refresh(db_order)
    
    logger.info(f"Order created: {db_order.id}, total: {total}")
    return db_order


@app.get("/orders", response_model=List[OrderResponse])
def get_orders(
    user_id: int = None,
    status: str = None,
    skip: int = 0,
    limit: int = 100,
    db: Session = Depends(get_db)
):
    """Получение заказов с фильтрацией."""
    query = db.query(Order)
    
    if user_id:
        query = query.filter(Order.user_id == user_id)
    if status:
        query = query.filter(Order.status == status)
    
    orders = query.offset(skip).limit(limit).all()
    return orders


@app.get("/orders/{order_id}", response_model=OrderResponse)
def get_order(order_id: int, db: Session = Depends(get_db)):
    """Получение заказа по ID."""
    order = db.query(Order).filter(Order.id == order_id).first()
    if not order:
        raise HTTPException(status_code=404, detail="Order not found")
    return order


# ============================================================
# ANALYTICS ENDPOINTS
# ============================================================

@app.get("/analytics/sales", response_model=List[SalesAnalyticsResponse])
def get_sales_analytics(
    start_date: str = None,
    end_date: str = None,
    db: Session = Depends(get_db)
):
    """Получение аналитики по продажам (данные из ETL)."""
    query = db.query(SalesAnalytics)
    
    if start_date:
        query = query.filter(SalesAnalytics.date >= start_date)
    if end_date:
        query = query.filter(SalesAnalytics.date <= end_date)
    
    analytics = query.order_by(SalesAnalytics.date).all()
    return analytics


@app.get("/analytics/summary")
def get_summary(db: Session = Depends(get_db)):
    """Получение общей сводки."""
    total_users = db.query(User).count()
    total_products = db.query(Product).count()
    total_orders = db.query(Order).count()
    total_revenue = db.query(db.func.sum(Order.total)).scalar() or 0
    
    return {
        "total_users": total_users,
        "total_products": total_products,
        "total_orders": total_orders,
        "total_revenue": float(total_revenue),
        "avg_order_value": float(total_revenue / total_orders) if total_orders else 0
    }
3.2. Сервисный слой
python
# services.py
from sqlalchemy.orm import Session
from sqlalchemy import func
from datetime import datetime, timedelta
from typing import List, Dict, Any
import logging

from models import Order, OrderItem, Product, SalesAnalytics

logger = logging.getLogger(__name__)


class OrderService:
    """Сервис для работы с заказами."""
    
    @staticmethod
    def calculate_order_total(db: Session, order_id: int) -> float:
        """Расчёт общей суммы заказа."""
        total = db.query(func.sum(OrderItem.price * OrderItem.quantity))\
                  .filter(OrderItem.order_id == order_id).scalar() or 0
        return float(total)
    
    @staticmethod
    def update_order_status(db: Session, order_id: int, status: str) -> bool:
        """Обновление статуса заказа."""
        order = db.query(Order).filter(Order.id == order_id).first()
        if not order:
            return False
        
        order.status = status
        db.commit()
        logger.info(f"Order {order_id} status updated to {status}")
        return True


class AnalyticsService:
    """Сервис для аналитики."""
    
    @staticmethod
    def get_daily_sales(db: Session, date: datetime) -> Dict:
        """Получение продаж за день."""
        start = datetime(date.year, date.month, date.day)
        end = start + timedelta(days=1)
        
        orders = db.query(Order).filter(
            Order.created_at >= start,
            Order.created_at < end,
            Order.status == 'completed'
        ).all()
        
        total_sales = sum(o.total for o in orders)
        order_count = len(orders)
        avg_order = total_sales / order_count if order_count else 0
        
        # Топ продукт дня
        top_product = db.query(
            Product.name,
            func.sum(OrderItem.quantity).label('total_quantity')
        ).join(OrderItem).join(Order).filter(
            Order.created_at >= start,
            Order.created_at < end,
            Order.status == 'completed'
        ).group_by(Product.id).order_by(func.sum(OrderItem.quantity).desc()).first()
        
        return {
            'date': start,
            'total_sales': float(total_sales),
            'order_count': order_count,
            'avg_order_value': float(avg_order),
            'top_product': top_product[0] if top_product else None,
            'top_product_quantity': top_product[1] if top_product else 0
        }
    
    @staticmethod
    def save_daily_analytics(db: Session, date: datetime):
        """Сохранение дневной аналитики в БД."""
        analytics = AnalyticsService.get_daily_sales(db, date)
        
        db_analytics = SalesAnalytics(
            date=analytics['date'],
            total_sales=analytics['total_sales'],
            order_count=analytics['order_count'],
            avg_order_value=analytics['avg_order_value']
        )
        
        # Находим ID топ продукта
        if analytics['top_product']:
            product = db.query(Product).filter(Product.name == analytics['top_product']).first()
            if product:
                db_analytics.top_product_id = product.id
        
        db.add(db_analytics)
        db.commit()
        logger.info(f"Analytics saved for {date.date()}")
4. Интеграция ETL-процесса
4.1. ETL модуль
python
# etl.py
import pandas as pd
from sqlalchemy.orm import Session
from datetime import datetime, timedelta
from typing import List, Dict
import logging

from models import Order, OrderItem, Product, SalesAnalytics
from services import AnalyticsService

logger = logging.getLogger(__name__)


class ETLProcessor:
    """ETL-процессор для обработки данных."""
    
    def __init__(self, db_session: Session):
        self.db = db_session
    
    def extract_orders(self, start_date: datetime, end_date: datetime) -> pd.DataFrame:
        """Извлечение данных о заказах за период."""
        orders = self.db.query(Order).filter(
            Order.created_at >= start_date,
            Order.created_at <= end_date,
            Order.status == 'completed'
        ).all()
        
        data = []
        for order in orders:
            for item in order.items:
                data.append({
                    'order_id': order.id,
                    'user_id': order.user_id,
                    'product_id': item.product_id,
                    'product_name': item.product.name,
                    'quantity': item.quantity,
                    'price': float(item.price),
                    'total': float(item.price * item.quantity),
                    'category': item.product.category,
                    'order_date': order.created_at.date()
                })
        
        df = pd.DataFrame(data)
        logger.info(f"Extracted {len(df)} order items")
        return df
    
    def transform_aggregate(self, df: pd.DataFrame) -> Dict:
        """Агрегация данных."""
        if df.empty:
            return {}
        
        # Общие метрики
        total_revenue = df['total'].sum()
        total_orders = df['order_id'].nunique()
        avg_order_value = total_revenue / total_orders if total_orders else 0
        
        # Топ продукты
        top_products = df.groupby('product_name')['quantity'].sum().sort_values(ascending=False).head(5)
        
        # Продажи по категориям
        sales_by_category = df.groupby('category')['total'].sum().sort_values(ascending=False)
        
        # Продажи по дням
        daily_sales = df.groupby('order_date')['total'].sum()
        
        return {
            'total_revenue': total_revenue,
            'total_orders': total_orders,
            'avg_order_value': avg_order_value,
            'top_products': top_products.to_dict(),
            'sales_by_category': sales_by_category.to_dict(),
            'daily_sales': daily_sales.to_dict(),
            'last_update': datetime.now()
        }
    
    def load_analytics(self, analytics: Dict):
        """Загрузка агрегированных данных в БД."""
        if not analytics:
            return
        
        # Сохранение в отдельную таблицу аналитики
        for date, sales in analytics.get('daily_sales', {}).items():
            existing = self.db.query(SalesAnalytics).filter(
                SalesAnalytics.date == date
            ).first()
            
            if not existing:
                db_analytics = SalesAnalytics(
                    date=datetime.strptime(str(date), '%Y-%m-%d'),
                    total_sales=sales,
                    order_count=analytics.get('total_orders', 0),
                    avg_order_value=analytics.get('avg_order_value', 0)
                )
                self.db.add(db_analytics)
        
        self.db.commit()
        logger.info(f"Loaded analytics for {len(analytics.get('daily_sales', {}))} days")
    
    def run_etl(self, days_back: int = 7):
        """Запуск полного ETL-процесса."""
        end_date = datetime.now()
        start_date = end_date - timedelta(days=days_back)
        
        logger.info(f"Starting ETL process for period {start_date.date()} to {end_date.date()}")
        
        # Extract
        df = self.extract_orders(start_date, end_date)
        
        # Transform
        analytics = self.transform_aggregate(df)
        
        # Load
        self.load_analytics(analytics)
        
        # Также сохраняем дневную аналитику
        for i in range(days_back):
            date = end_date - timedelta(days=i)
            AnalyticsService.save_daily_analytics(self.db, date)
        
        logger.info("ETL process completed")
        return analytics


class ETLTaskScheduler:
    """Планировщик ETL-задач."""
    
    def __init__(self, db_session_factory):
        self.db_session_factory = db_session_factory
        self.is_running = False
    
    def run_once(self):
        """Однократный запуск ETL."""
        db = self.db_session_factory()
        try:
            etl = ETLProcessor(db)
            result = etl.run_etl(days_back=1)
            return result
        finally:
            db.close()
    
    def run_daily(self):
        """Ежедневный запуск ETL."""
        import time
        self.is_running = True
        
        while self.is_running:
            # Ждём до следующей полуночи
            now = datetime.now()
            tomorrow = now + timedelta(days=1)
            midnight = datetime(tomorrow.year, tomorrow.month, tomorrow.day, 0, 0, 0)
            wait_seconds = (midnight - now).total_seconds()
            
            logger.info(f"Next ETL run in {wait_seconds / 3600:.1f} hours")
            time.sleep(wait_seconds)
            
            # Запускаем ETL
            self.run_once()
    
    def stop(self):
        """Остановка планировщика."""
        self.is_running = False
5. Полная интеграция
5.1. Конфигурация и инициализация
python
# app.py
from fastapi import FastAPI, Depends
from contextlib import asynccontextmanager
import threading
import logging

from database import engine, Base, get_db, SessionLocal
from etl import ETLTaskScheduler
import main as api

# Настройка логирования
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Глобальные переменные
etl_scheduler = None
etl_thread = None


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Управление жизненным циклом приложения."""
    global etl_scheduler, etl_thread
    
    # Запуск
    logger.info("Starting application...")
    
    # Создание таблиц
    Base.metadata.create_all(bind=engine)
    
    # Запуск ETL-планировщика в отдельном потоке
    etl_scheduler = ETLTaskScheduler(SessionLocal)
    etl_thread = threading.Thread(target=etl_scheduler.run_daily, daemon=True)
    etl_thread.start()
    logger.info("ETL scheduler started")
    
    yield
    
    # Остановка
    logger.info("Shutting down application...")
    if etl_scheduler:
        etl_scheduler.stop()
    
    logger.info("Application stopped")


# Создание приложения
app = FastAPI(
    title="E-Commerce Platform",
    description="Интегрированная система: API + БД + ETL",
    version="1.0.0",
    lifespan=lifespan
)

# Подключение API эндпоинтов
app.include_router(api.router)


# Дополнительный эндпоинт для ручного запуска ETL
@app.post("/etl/run")
def run_etl_manual():
    """Ручной запуск ETL-процесса."""
    db = SessionLocal()
    try:
        from etl import ETLProcessor
        etl = ETLProcessor(db)
        result = etl.run_etl(days_back=1)
        return {"status": "success", "result": result}
    except Exception as e:
        logger.error(f"ETL failed: {e}")
        return {"status": "error", "message": str(e)}
    finally:
        db.close()


@app.get("/health")
def health_check():
    """Проверка здоровья системы."""
    db = SessionLocal()
    try:
        db.execute("SELECT 1")
        db_status = "ok"
    except Exception:
        db_status = "error"
    finally:
        db.close()
    
    return {
        "status": "healthy",
        "database": db_status,
        "etl_scheduler": "running" if etl_scheduler and etl_scheduler.is_running else "stopped"
    }
5.2. Загрузка тестовых данных
python
# init_data.py
"""
Скрипт для инициализации базы данных тестовыми данными.
"""

from database import SessionLocal
from models import User, Product, Order, OrderItem
from datetime import datetime, timedelta
import random


def init_test_data():
    """Заполнение БД тестовыми данными."""
    db = SessionLocal()
    
    try:
        # Очистка старых данных
        db.query(OrderItem).delete()
        db.query(Order).delete()
        db.query(Product).delete()
        db.query(User).delete()
        
        # Создание пользователей
        users = []
        for i in range(10):
            user = User(
                name=f"User_{i}",
                email=f"user_{i}@example.com"
            )
            db.add(user)
            users.append(user)
        db.flush()
        
        # Создание товаров
        products = []
        categories = ['electronics', 'clothing', 'books', 'sports', 'toys']
        for i in range(20):
            product = Product(
                name=f"Product_{i}",
                price=random.uniform(10, 1000),
                stock=random.randint(0, 100),
                category=random.choice(categories)
            )
            db.add(product)
            products.append(product)
        db.flush()
        
        # Создание заказов за последние 30 дней
        for day in range(30):
            date = datetime.now() - timedelta(days=day)
            
            for _ in range(random.randint(1, 5)):
                user = random.choice(users)
                order = Order(
                    user_id=user.id,
                    status=random.choice(['pending', 'completed', 'cancelled']),
                    created_at=date.replace(hour=random.randint(8, 20), minute=random.randint(0, 59))
                )
                db.add(order)
                db.flush()
                
                total = 0
                for _ in range(random.randint(1, 3)):
                    product = random.choice(products)
                    quantity = random.randint(1, 3)
                    
                    if product.stock >= quantity:
                        product.stock -= quantity
                        order_item = OrderItem(
                            order_id=order.id,
                            product_id=product.id,
                            quantity=quantity,
                            price=product.price
                        )
                        db.add(order_item)
                        total += product.price * quantity
                
                order.total = total
        
        db.commit()
        print("✅ Test data loaded successfully")
        
    except Exception as e:
        db.rollback()
        print(f"❌ Error: {e}")
    finally:
        db.close()


if __name__ == "__main__":
    init_test_data()
6. Практические примеры
6.1. CLI-клиент для взаимодействия с API
python
# client.py
import requests
import json
from typing import Optional


class APIClient:
    """Клиент для работы с API."""
    
    def __init__(self, base_url: str = "http://localhost:8000"):
        self.base_url = base_url
    
    def create_user(self, name: str, email: str) -> dict:
        response = requests.post(
            f"{self.base_url}/users",
            json={"name": name, "email": email}
        )
        response.raise_for_status()
        return response.json()
    
    def get_users(self) -> list:
        response = requests.get(f"{self.base_url}/users")
        response.raise_for_status()
        return response.json()
    
    def create_product(self, name: str, price: float, stock: int, category: str = None) -> dict:
        data = {"name": name, "price": price, "stock": stock}
        if category:
            data["category"] = category
        response = requests.post(f"{self.base_url}/products", json=data)
        response.raise_for_status()
        return response.json()
    
    def create_order(self, user_id: int, items: list) -> dict:
        response = requests.post(
            f"{self.base_url}/orders/{user_id}",
            json={"items": items}
        )
        response.raise_for_status()
        return response.json()
    
    def get_analytics(self) -> dict:
        response = requests.get(f"{self.base_url}/analytics/summary")
        response.raise_for_status()
        return response.json()
    
    def run_etl(self) -> dict:
        response = requests.post(f"{self.base_url}/etl/run")
        response.raise_for_status()
        return response.json()


def main():
    """Демонстрация работы клиента."""
    client = APIClient()
    
    print("=" * 60)
    print("СОЗДАНИЕ ТЕСТОВЫХ ДАННЫХ")
    print("=" * 60)
    
    # Создание пользователя
    user = client.create_user("Иван Петров", "ivan@example.com")
    print(f"✅ Создан пользователь: {user['name']} (ID: {user['id']})")
    
    # Создание товаров
    product1 = client.create_product("Ноутбук", 50000, 10, "electronics")
    product2 = client.create_product("Мышь", 1500, 50, "electronics")
    print(f"✅ Созданы товары: {product1['name']}, {product2['name']}")
    
    # Создание заказа
    order = client.create_order(user['id'], [
        {"product_id": product1['id'], "quantity": 1},
        {"product_id": product2['id'], "quantity": 2}
    ])
    print(f"✅ Создан заказ #{order['id']} на сумму {order['total']} ₽")
    
    # Получение аналитики
    analytics = client.get_analytics()
    print(f"\n📊 Аналитика:")
    print(f"   Всего пользователей: {analytics['total_users']}")
    print(f"   Всего товаров: {analytics['total_products']}")
    print(f"   Всего заказов: {analytics['total_orders']}")
    print(f"   Общая выручка: {analytics['total_revenue']} ₽")
    
    # Запуск ETL
    print("\n🔄 Запуск ETL-процесса...")
    result = client.run_etl()
    print(f"   Статус: {result['status']}")


if __name__ == "__main__":
    main()
7. Контрольные вопросы
Какие компоненты входят в интегрированную систему?

Как организовано взаимодействие между API, БД и ETL?

Зачем нужен сервисный слой между API и БД?

Как ETL-процесс получает данные из БД?

Как обрабатываются фоновые задачи (ETL) в FastAPI?

Какие типы связей используются между таблицами?

Как обеспечить целостность данных при создании заказа?

Зачем нужна отдельная таблица для аналитики?

Как протестировать интеграцию всех компонентов?

Как масштабировать такую систему?

8. Практическое задание
Задание 1 (базовое)
Создайте простое API с одной моделью (например, Task) и подключите к нему SQLite.

Задание 2 (среднее)
Добавьте ETL-процесс, который ежедневно агрегирует данные и сохраняет в отдельную таблицу.

Задание 3 (сложное)
Разработайте полную интегрированную систему для интернет-магазина:

API для управления пользователями, товарами, заказами

БД PostgreSQL с миграциями

ETL для дневной агрегации продаж

Эндпоинты для получения аналитики

Планировщик ETL-задач

9. Шпаргалка
python
# === АРХИТЕКТУРА ===
# API (FastAPI) <-> Service Layer <-> БД (SQLAlchemy)
# ETL <-> БД (прямой доступ)

# === КЛЮЧЕВЫЕ МОМЕНТЫ ===
# 1. Используйте зависимости для сессий БД
# 2. Сервисный слой для бизнес-логики
# 3. ETL в отдельном потоке
# 4. Валидация через Pydantic
# 5. Логирование всех операций

# === ПОЛЕЗНЫЕ КОМАНДЫ ===
# Запуск API: uvicorn app:app --reload
# Инициализация БД: python init_data.py
# Запуск ETL вручную: POST /etl/run
# Проверка здоровья: GET /health
Итог лекции
Вы сегодня:

✅ Изучили архитектуру интегрированной системы API + БД + ETL

✅ Реализовали полноценное API на FastAPI с подключением к БД

✅ Добавили ETL-процесс для агрегации данных

✅ Интегрировали все компоненты в единое приложение

✅ Создали клиент для взаимодействия с API

Теперь вы можете создавать полноценные, масштабируемые приложения с интеграцией всех ключевых компонентов!

