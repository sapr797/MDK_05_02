# Тема 2.41. Централизованная обработка ошибок и кастомные исключения в FastAPI

**Цель лекции:**  
Изучить механизмы централизованной обработки ошибок в FastAPI, создание кастомных исключений, глобальных обработчиков и middleware для логирования ошибок.

> Главная мысль: **Хорошее приложение не просто падает с ошибкой — оно даёт понятный ответ, логирует проблему и продолжает работать для остальных пользователей.**

---

## Содержание

1. [Введение в обработку ошибок](#1-введение-в-обработку-ошибок)
2. [Кастомные исключения](#2-кастомные-исключения)
3. [Глобальные обработчики исключений](#3-глобальные-обработчики-исключений)
4. [Middleware для обработки ошибок](#4-middleware-для-обработки-ошибок)
5. [Логирование ошибок](#5-логирование-ошибок)
6. [Практические примеры](#6-практические-примеры)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Введение в обработку ошибок

### 1.1. Проблемы без централизованной обработки
┌─────────────────────────────────────────────────────────────────────────────┐
│ ПРОБЛЕМЫ ДЕЦЕНТРАЛИЗОВАННОЙ ОБРАБОТКИ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ❌ БЕЗ ЦЕНТРАЛИЗАЦИИ: │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ @app.get("/user/{id}") │ │
│ │ def get_user(id: int): │ │
│ │ try: │ │
│ │ user = db.query(User).filter_by(id=id).first() │ │
│ │ if not user: │ │
│ │ return JSONResponse(status_code=404, ...) │ │
│ │ except DatabaseError as e: │ │
│ │ return JSONResponse(status_code=500, ...) │ │
│ │ except ValidationError as e: │ │
│ │ return JSONResponse(status_code=422, ...) │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
│ Проблемы: │
│ • Дублирование кода обработки ошибок │
│ • Непоследовательные форматы ответов │
│ • Пропущенные обработчики │
│ • Сложность поддержки │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Иерархия обработки ошибок в FastAPI
┌─────────────────────────────────────────────────────────────────────────────┐
│ ИЕРАРХИЯ ОБРАБОТКИ ОШИБОК │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ 1. БЛИЖАЙШИЙ ОБРАБОТЧИК (try/except в эндпоинте) │
│ └── Локальная обработка специфичных ошибок │
│ │
│ 2. ГЛОБАЛЬНЫЙ ОБРАБОТЧИК (@app.exception_handler) │
│ └── Обработка конкретных типов исключений │
│ │
│ 3. MIDDLEWARE │
│ └── Перехват всех исключений на уровне запроса │
│ │
│ 4. ВСТРОЕННЫЕ ОБРАБОТЧИКИ FASTAPI │
│ └── HTTPException, RequestValidationError │
│ │
│ 5. ПОСЛЕДНИЙ РУБЕЖ (ASGI сервер) │
│ └── 500 Internal Server Error │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

---

## 2. Кастомные исключения

### 2.1. Базовое кастомное исключение

```python
from fastapi import HTTPException, status
from typing import Any, Dict, Optional


class AppException(HTTPException):
    """Базовое исключение приложения."""
    
    def __init__(
        self,
        status_code: int,
        message: str,
        error_code: str = None,
        details: Any = None
    ):
        super().__init__(status_code=status_code, detail=message)
        self.message = message
        self.error_code = error_code
        self.details = details


class NotFoundError(AppException):
    """Ресурс не найден."""
    
    def __init__(self, resource: str, identifier: Any):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            message=f"{resource} с идентификатором '{identifier}' не найден",
            error_code="RESOURCE_NOT_FOUND",
            details={"resource": resource, "identifier": identifier}
        )


class ValidationError(AppException):
    """Ошибка валидации."""
    
    def __init__(self, field: str, message: str, value: Any = None):
        super().__init__(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
            message=f"Ошибка валидации поля '{field}': {message}",
            error_code="VALIDATION_ERROR",
            details={"field": field, "value": value, "message": message}
        )


class BusinessError(AppException):
    """Ошибка бизнес-логики."""
    
    def __init__(self, message: str, error_code: str = "BUSINESS_ERROR"):
        super().__init__(
            status_code=status.HTTP_400_BAD_REQUEST,
            message=message,
            error_code=error_code
        )


class UnauthorizedError(AppException):
    """Ошибка авторизации."""
    
    def __init__(self, message: str = "Требуется авторизация"):
        super().__init__(
            status_code=status.HTTP_401_UNAUTHORIZED,
            message=message,
            error_code="UNAUTHORIZED"
        )


class ForbiddenError(AppException):
    """Доступ запрещён."""
    
    def __init__(self, message: str = "Недостаточно прав"):
        super().__init__(
            status_code=status.HTTP_403_FORBIDDEN,
            message=message,
            error_code="FORBIDDEN"
        )
2.2. Специализированные исключения для предметной области
python
# domain_exceptions.py
from typing import Any, Optional


class UserNotFoundError(NotFoundError):
    """Пользователь не найден."""
    
    def __init__(self, user_id: int):
        super().__init__("Пользователь", user_id)
        self.error_code = "USER_NOT_FOUND"


class ProductNotFoundError(NotFoundError):
    """Товар не найден."""
    
    def __init__(self, product_id: int):
        super().__init__("Товар", product_id)
        self.error_code = "PRODUCT_NOT_FOUND"


class OrderNotFoundError(NotFoundError):
    """Заказ не найден."""
    
    def __init__(self, order_id: int):
        super().__init__("Заказ", order_id)
        self.error_code = "ORDER_NOT_FOUND"


class InsufficientStockError(BusinessError):
    """Недостаточно товара на складе."""
    
    def __init__(self, product_name: str, requested: int, available: int):
        super().__init__(
            message=f"Недостаточно товара '{product_name}'. Запрошено: {requested}, доступно: {available}",
            error_code="INSUFFICIENT_STOCK"
        )
        self.product_name = product_name
        self.requested = requested
        self.available = available


class DuplicateEmailError(BusinessError):
    """Email уже зарегистрирован."""
    
    def __init__(self, email: str):
        super().__init__(
            message=f"Пользователь с email '{email}' уже существует",
            error_code="DUPLICATE_EMAIL"
        )


class InvalidOrderStatusError(BusinessError):
    """Неверный статус заказа для операции."""
    
    def __init__(self, order_id: int, current_status: str, required_statuses: list):
        super().__init__(
            message=f"Заказ #{order_id} имеет статус '{current_status}', требуется: {required_statuses}",
            error_code="INVALID_ORDER_STATUS"
        )


class PaymentError(BusinessError):
    """Ошибка оплаты."""
    
    def __init__(self, message: str, payment_id: Optional[str] = None):
        super().__init__(
            message=f"Ошибка оплаты: {message}",
            error_code="PAYMENT_ERROR"
        )
        self.payment_id = payment_id


class DatabaseError(AppException):
    """Ошибка базы данных."""
    
    def __init__(self, message: str, original_error: Optional[Exception] = None):
        super().__init__(
            status_code=500,
            message=f"Ошибка базы данных: {message}",
            error_code="DATABASE_ERROR",
            details={"original_error": str(original_error) if original_error else None}
        )
3. Глобальные обработчики исключений
3.1. Обработчик для кастомных исключений
python
# exception_handlers.py
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from starlette.exceptions import HTTPException as StarletteHTTPException
import logging

from exceptions import AppException

logger = logging.getLogger(__name__)


def register_exception_handlers(app: FastAPI):
    """Регистрация всех глобальных обработчиков."""
    
    @app.exception_handler(AppException)
    async def app_exception_handler(request: Request, exc: AppException):
        """Обработчик кастомных исключений приложения."""
        logger.warning(
            f"AppException: {exc.error_code} - {exc.message}",
            extra={
                "status_code": exc.status_code,
                "path": request.url.path,
                "method": request.method
            }
        )
        
        return JSONResponse(
            status_code=exc.status_code,
            content={
                "error": True,
                "code": exc.error_code,
                "message": exc.message,
                "details": exc.details,
                "path": request.url.path
            }
        )
    
    @app.exception_handler(StarletteHTTPException)
    async def http_exception_handler(request: Request, exc: StarletteHTTPException):
        """Обработчик HTTP исключений."""
        logger.warning(
            f"HTTPException: {exc.status_code} - {exc.detail}",
            extra={"path": request.url.path, "method": request.method}
        )
        
        return JSONResponse(
            status_code=exc.status_code,
            content={
                "error": True,
                "code": "HTTP_ERROR",
                "message": exc.detail,
                "path": request.url.path
            }
        )
    
    @app.exception_handler(RequestValidationError)
    async def validation_exception_handler(request: Request, exc: RequestValidationError):
        """Обработчик ошибок валидации Pydantic."""
        errors = []
        for error in exc.errors():
            errors.append({
                "field": ".".join(str(loc) for loc in error['loc']),
                "message": error['msg'],
                "type": error['type']
            })
        
        logger.warning(
            f"Validation error: {len(errors)} errors",
            extra={"path": request.url.path, "method": request.method, "errors": errors}
        )
        
        return JSONResponse(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
            content={
                "error": True,
                "code": "VALIDATION_ERROR",
                "message": "Ошибка валидации данных",
                "details": errors,
                "path": request.url.path
            }
        )
    
    @app.exception_handler(Exception)
    async def general_exception_handler(request: Request, exc: Exception):
        """Обработчик всех непредвиденных исключений."""
        logger.error(
            f"Unhandled exception: {type(exc).__name__} - {str(exc)}",
            extra={"path": request.url.path, "method": request.method},
            exc_info=True
        )
        
        return JSONResponse(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            content={
                "error": True,
                "code": "INTERNAL_SERVER_ERROR",
                "message": "Внутренняя ошибка сервера",
                "path": request.url.path
            }
        )
3.2. Применение в приложении
python
# main.py
from fastapi import FastAPI
from exception_handlers import register_exception_handlers
from exceptions import UserNotFoundError, InsufficientStockError, DuplicateEmailError

app = FastAPI()

# Регистрация обработчиков
register_exception_handlers(app)


# Использование кастомных исключений в эндпоинтах
@app.get("/users/{user_id}")
def get_user(user_id: int):
    """Получение пользователя."""
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise UserNotFoundError(user_id)
    return user


@app.post("/orders")
def create_order(order_data: dict):
    """Создание заказа."""
    # Проверка остатков
    product = db.query(Product).filter(Product.id == order_data['product_id']).first()
    if not product:
        raise ProductNotFoundError(order_data['product_id'])
    
    if product.stock < order_data['quantity']:
        raise InsufficientStockError(
            product_name=product.name,
            requested=order_data['quantity'],
            available=product.stock
        )
    
    # Создание заказа...
    return {"status": "success"}


@app.post("/users")
def create_user(user_data: dict):
    """Создание пользователя."""
    existing = db.query(User).filter(User.email == user_data['email']).first()
    if existing:
        raise DuplicateEmailError(user_data['email'])
    
    # Создание пользователя...
    return {"status": "success"}
4. Middleware для обработки ошибок
4.1. Error Logging Middleware
python
# middleware.py
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware
import time
import logging
import uuid
from typing import Callable

logger = logging.getLogger(__name__)


class ErrorLoggingMiddleware(BaseHTTPMiddleware):
    """Middleware для логирования ошибок и добавления request_id."""
    
    async def dispatch(self, request: Request, call_next: Callable):
        # Генерация request_id
        request_id = str(uuid.uuid4())[:8]
        request.state.request_id = request_id
        
        start_time = time.time()
        
        try:
            response = await call_next(request)
            
            # Добавляем request_id в заголовки ответа
            response.headers["X-Request-ID"] = request_id
            
            # Логирование успешных запросов
            elapsed = time.time() - start_time
            logger.info(
                f"Request completed",
                extra={
                    "request_id": request_id,
                    "method": request.method,
                    "path": request.url.path,
                    "status_code": response.status_code,
                    "duration_ms": round(elapsed * 1000, 2)
                }
            )
            
            return response
            
        except Exception as e:
            # Логирование ошибки с контекстом
            elapsed = time.time() - start_time
            logger.error(
                f"Request failed: {type(e).__name__} - {str(e)}",
                extra={
                    "request_id": request_id,
                    "method": request.method,
                    "path": request.url.path,
                    "duration_ms": round(elapsed * 1000, 2)
                },
                exc_info=True
            )
            raise


class RequestContextMiddleware(BaseHTTPMiddleware):
    """Middleware для добавления контекста запроса."""
    
    async def dispatch(self, request: Request, call_next: Callable):
        # Добавляем информацию о запросе
        request.state.client_ip = request.client.host if request.client else "unknown"
        request.state.user_agent = request.headers.get("user-agent", "unknown")
        
        # Добавляем метку времени
        request.state.start_time = time.time()
        
        response = await call_next(request)
        
        # Добавляем время выполнения в заголовок
        elapsed = time.time() - request.state.start_time
        response.headers["X-Response-Time"] = f"{elapsed:.3f}s"
        
        return response
4.2. Подключение middleware
python
# main.py
from fastapi import FastAPI
from middleware import ErrorLoggingMiddleware, RequestContextMiddleware

app = FastAPI()

# Добавление middleware (порядок важен!)
app.add_middleware(RequestContextMiddleware)
app.add_middleware(ErrorLoggingMiddleware)
5. Логирование ошибок
5.1. Структурированное логирование ошибок
python
# error_logger.py
import logging
import json
from datetime import datetime
from typing import Any, Dict, Optional
from contextvars import ContextVar

# Контекстные переменные для передачи request_id
request_id_var: ContextVar[str] = ContextVar("request_id", default="unknown")


class StructuredLogger:
    """Структурированное логирование ошибок."""
    
    def __init__(self, name: str = "app"):
        self.logger = logging.getLogger(name)
    
    def _get_request_id(self) -> str:
        """Получение текущего request_id."""
        return request_id_var.get()
    
    def _format_log(self, level: str, message: str, **kwargs) -> Dict:
        """Форматирование лога."""
        return {
            "timestamp": datetime.now().isoformat(),
            "level": level,
            "request_id": self._get_request_id(),
            "message": message,
            **kwargs
        }
    
    def error(
        self,
        message: str,
        error_type: Optional[str] = None,
        error_code: Optional[str] = None,
        details: Optional[Dict] = None,
        exc_info: bool = False
    ):
        """Логирование ошибки."""
        log_entry = self._format_log(
            "ERROR",
            message,
            error_type=error_type,
            error_code=error_code,
            details=details
        )
        self.logger.error(json.dumps(log_entry, ensure_ascii=False), exc_info=exc_info)
    
    def warning(self, message: str, **kwargs):
        """Логирование предупреждения."""
        log_entry = self._format_log("WARNING", message, **kwargs)
        self.logger.warning(json.dumps(log_entry, ensure_ascii=False))
    
    def info(self, message: str, **kwargs):
        """Информационное логирование."""
        log_entry = self._format_log("INFO", message, **kwargs)
        self.logger.info(json.dumps(log_entry, ensure_ascii=False))
    
    def debug(self, message: str, **kwargs):
        """Отладочное логирование."""
        log_entry = self._format_log("DEBUG", message, **kwargs)
        self.logger.debug(json.dumps(log_entry, ensure_ascii=False))


# Глобальный экземпляр
error_logger = StructuredLogger()


def log_error(
    error: Exception,
    context: Optional[Dict] = None,
    request_id: Optional[str] = None
):
    """Утилита для логирования ошибок."""
    error_logger.error(
        message=str(error),
        error_type=type(error).__name__,
        details=context
    )
5.2. Интеграция с FastAPI через зависимости
python
# dependencies.py
from fastapi import Request
from contextvars import ContextVar


async def set_request_context(request: Request):
    """Установка контекста запроса для логирования."""
    request_id = getattr(request.state, "request_id", "unknown")
    request_id_var.set(request_id)
    return {"request_id": request_id}


# В main.py
from dependencies import set_request_context

app = FastAPI()

@app.middleware("http")
async def set_context_middleware(request: Request, call_next):
    request_id_var.set(getattr(request.state, "request_id", "unknown"))
    return await call_next(request)
6. Практические примеры
6.1. Полная система обработки ошибок
python
# complete_error_handling.py
"""
Полная система централизованной обработки ошибок для FastAPI.
"""

from fastapi import FastAPI, Request, Depends
from fastapi.responses import JSONResponse
from fastapi.security import HTTPBearer
from typing import Optional, Dict, Any
import time
import uuid
import logging

# Настройка логирования
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


# ============================================================
# КАСТОМНЫЕ ИСКЛЮЧЕНИЯ
# ============================================================

class APIError(Exception):
    """Базовое исключение API."""
    
    def __init__(
        self,
        status_code: int,
        message: str,
        error_code: str = "API_ERROR",
        details: Any = None
    ):
        self.status_code = status_code
        self.message = message
        self.error_code = error_code
        self.details = details
        super().__init__(message)


class NotFoundError(APIError):
    def __init__(self, resource: str, identifier: Any):
        super().__init__(
            status_code=404,
            message=f"{resource} '{identifier}' не найден",
            error_code="NOT_FOUND"
        )


class ValidationError(APIError):
    def __init__(self, errors: list):
        super().__init__(
            status_code=422,
            message="Ошибка валидации",
            error_code="VALIDATION_ERROR",
            details=errors
        )


class UnauthorizedError(APIError):
    def __init__(self):
        super().__init__(
            status_code=401,
            message="Требуется авторизация",
            error_code="UNAUTHORIZED"
        )


class RateLimitError(APIError):
    def __init__(self, retry_after: int):
        super().__init__(
            status_code=429,
            message="Слишком много запросов",
            error_code="RATE_LIMIT_EXCEEDED",
            details={"retry_after": retry_after}
        )


# ============================================================
# ГЛОБАЛЬНЫЕ ОБРАБОТЧИКИ
# ============================================================

def register_handlers(app: FastAPI):
    """Регистрация всех обработчиков ошибок."""
    
    @app.exception_handler(APIError)
    async def api_error_handler(request: Request, exc: APIError):
        logger.warning(
            f"API Error: {exc.error_code} - {exc.message}",
            extra={"path": request.url.path}
        )
        
        return JSONResponse(
            status_code=exc.status_code,
            content={
                "success": False,
                "error": {
                    "code": exc.error_code,
                    "message": exc.message,
                    "details": exc.details
                }
            }
        )
    
    @app.exception_handler(Exception)
    async def general_handler(request: Request, exc: Exception):
        logger.error(
            f"Unhandled error: {type(exc).__name__} - {str(exc)}",
            exc_info=True,
            extra={"path": request.url.path}
        )
        
        return JSONResponse(
            status_code=500,
            content={
                "success": False,
                "error": {
                    "code": "INTERNAL_ERROR",
                    "message": "Внутренняя ошибка сервера"
                }
            }
        )


# ============================================================
# MIDDLEWARE
# ============================================================

class RequestIDMiddleware:
    """Добавление request_id к каждому запросу."""
    
    async def __call__(self, request: Request, call_next):
        request_id = request.headers.get("X-Request-ID", str(uuid.uuid4())[:8])
        request.state.request_id = request_id
        
        response = await call_next(request)
        response.headers["X-Request-ID"] = request_id
        
        return response


class PerformanceMiddleware:
    """Измерение времени выполнения запросов."""
    
    async def __call__(self, request: Request, call_next):
        start_time = time.time()
        response = await call_next(request)
        elapsed = (time.time() - start_time) * 1000
        
        response.headers["X-Response-Time"] = f"{elapsed:.2f}ms"
        
        if elapsed > 1000:
            logger.warning(f"Slow request: {request.method} {request.url.path} - {elapsed:.2f}ms")
        
        return response


# ============================================================
# ПРИМЕР ИСПОЛЬЗОВАНИЯ
# ============================================================

app = FastAPI(title="Error Handling Demo")

# Добавление middleware
app.add_middleware(RequestIDMiddleware)
app.add_middleware(PerformanceMiddleware)

# Регистрация обработчиков
register_handlers(app)

# Хранилище данных
users_db = {}


# Эндпоинты с использованием кастомных исключений
@app.post("/users/{user_id}")
def create_user(user_id: int, name: str, email: str):
    """Создание пользователя."""
    if user_id in users_db:
        raise APIError(409, f"User {user_id} already exists", "DUPLICATE_ID")
    
    if "@" not in email:
        raise ValidationError([{"field": "email", "message": "Invalid email format"}])
    
    users_db[user_id] = {"name": name, "email": email}
    return {"success": True, "data": users_db[user_id]}


@app.get("/users/{user_id}")
def get_user(user_id: int):
    """Получение пользователя."""
    if user_id not in users_db:
        raise NotFoundError("User", user_id)
    
    return {"success": True, "data": users_db[user_id]}


@app.get("/error")
def generate_error():
    """Генерация ошибки для демонстрации."""
    raise ValueError("Something went wrong")


@app.get("/protected")
def protected_route(token: str = Depends(HTTPBearer())):
    """Защищённый маршрут."""
    if not token.credentials == "secret":
        raise UnauthorizedError()
    
    return {"success": True, "message": "Access granted"}


@app.get("/slow")
async def slow_endpoint():
    """Медленный эндпоинт для демонстрации PerformanceMiddleware."""
    import asyncio
    await asyncio.sleep(1.5)
    return {"success": True, "message": "Completed"}


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000, reload=True)
6.2. Тестирование обработки ошибок
python
# test_error_handling.py
import pytest
from fastapi.testclient import TestClient
from complete_error_handling import app

client = TestClient(app)


def test_not_found_error():
    """Тест ошибки 404."""
    response = client.get("/users/999")
    assert response.status_code == 404
    assert response.json()["error"]["code"] == "NOT_FOUND"


def test_validation_error():
    """Тест ошибки валидации."""
    response = client.post("/users/1?name=Test&email=invalid")
    assert response.status_code == 422
    assert response.json()["error"]["code"] == "VALIDATION_ERROR"


def test_duplicate_error():
    """Тест ошибки дубликата."""
    client.post("/users/1?name=Test&email=test@example.com")
    response = client.post("/users/1?name=Test2&email=test2@example.com")
    assert response.status_code == 409
    assert response.json()["error"]["code"] == "DUPLICATE_ID"


def test_general_error():
    """Тест общей ошибки."""
    response = client.get("/error")
    assert response.status_code == 500
    assert response.json()["error"]["code"] == "INTERNAL_ERROR"


def test_unauthorized_error():
    """Тест ошибки авторизации."""
    response = client.get("/protected")
    assert response.status_code == 401
    assert response.json()["error"]["code"] == "UNAUTHORIZED"


def test_request_id_header():
    """Тест наличия request_id в заголовках."""
    response = client.get("/users/1")
    assert "X-Request-ID" in response.headers


def test_response_time_header():
    """Тест наличия времени ответа в заголовках."""
    response = client.get("/users/1")
    assert "X-Response-Time" in response.headers
7. Контрольные вопросы
Почему важна централизованная обработка ошибок?

Как создать кастомное исключение в FastAPI?

В чём разница между @app.exception_handler и middleware?

Как добавить request_id к каждому запросу?

Как логировать ошибки с контекстом запроса?

Как обработать ошибки валидации Pydantic?

Как вернуть единый формат ошибки для всех эндпоинтов?

Как измерить время выполнения запроса?

Как отличить ожидаемые ошибки от непредвиденных?

Как протестировать обработку ошибок?

8. Практическое задание
Задание 1 (базовое)
Создайте кастомное исключение TaskNotFoundError и глобальный обработчик для него.

Задание 2 (среднее)
Добавьте middleware для логирования всех ошибок с request_id и временем выполнения.

Задание 3 (сложное)
Реализуйте полную систему обработки ошибок для интернет-магазина:

Кастомные исключения для каждого ресурса (User, Product, Order)

Глобальные обработчики с единым форматом ответа

Логирование в JSON-формате

Middleware для метрик

9. Шпаргалка
python
# === КАСТОМНОЕ ИСКЛЮЧЕНИЕ ===
class MyError(Exception):
    def __init__(self, message: str):
        self.message = message
        super().__init__(message)

# === ГЛОБАЛЬНЫЙ ОБРАБОТЧИК ===
@app.exception_handler(MyError)
async def my_error_handler(request: Request, exc: MyError):
    return JSONResponse(
        status_code=400,
        content={"error": exc.message}
    )

# === MIDDLEWARE ===
class MyMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        try:
            return await call_next(request)
        except Exception as e:
            # обработка ошибки
            raise

# === ЛОГИРОВАНИЕ ===
logger.error("Error occurred", exc_info=True)

# === ЕДИНЫЙ ФОРМАТ ОТВЕТА ===
{
    "success": False,
    "error": {
        "code": "ERROR_CODE",
        "message": "Error description",
        "details": {}
    }
}
Итог лекции
Вы сегодня:

✅ Изучили принципы централизованной обработки ошибок

✅ Создали иерархию кастомных исключений

✅ Реализовали глобальные обработчики для разных типов ошибок

✅ Добавили middleware для логирования и метрик

✅ Создали единый формат ответов об ошибках

Теперь ваше API даёт понятные ответы об ошибках и логирует проблемы для быстрого исправления!

