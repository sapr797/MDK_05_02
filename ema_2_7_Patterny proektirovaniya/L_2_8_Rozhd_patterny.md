# Тема 2.8. Порождающие паттерны: Factory Method, Singleton. Синтаксис и стиль кода

**Цель лекции:**  
Изучить порождающие паттерны проектирования Factory Method и Singleton, понять их применение в Python, освоить правильный синтаксис и стиль реализации, научиться выбирать подходящий паттерн для решения задач.

> Главная мысль: **Паттерны проектирования — это проверенные решения типовых проблем. Не нужно изобретать велосипед, когда есть готовый, отлаженный механизм.**

---

## Часть 1. Введение в паттерны проектирования

### 1.1. Что такое паттерны проектирования

**Паттерн проектирования** — это типовое, проверенное решение часто встречающейся проблемы в объектно-ориентированном дизайне.

**Категории паттернов (по GoF — Gang of Four):**

| Категория | Назначение | Примеры |
|-----------|------------|---------|
| **Порождающие** | Создание объектов | Factory Method, Singleton, Abstract Factory, Builder, Prototype |
| **Структурные** | Компоновка классов и объектов | Adapter, Decorator, Facade, Proxy |
| **Поведенческие** | Взаимодействие объектов | Observer, Strategy, Command, Template Method |

### 1.2. Зачем нужны порождающие паттерны

**Проблемы, которые решают порождающие паттерны:**

1. **Жёсткая привязка к конкретным классам** — код зависит от конкретных реализаций
2. **Сложная логика создания** — создание объекта требует множества шагов
3. **Неуправляемое количество объектов** — трудно контролировать, сколько объектов создаётся
4. **Дублирование кода создания** — одна и та же логика повторяется

**Преимущества использования:**
Без паттернов С паттернами
─────────────────────────────────────────────────────
Код знает конкретные классы Код работает с интерфейсами
Изменение требует правок везде Изменение только в одном месте
Трудно тестировать Легко подменять моками
Сложно расширять Легко добавлять новые типы

text

---

## Часть 2. Паттерн Factory Method (Фабричный метод)

### 2.1. Назначение и проблема

**Назначение:** Определяет интерфейс для создания объекта, но оставляет подклассам решение о том, какой класс инстанцировать.

**Проблема:** Как сделать так, чтобы код создания объектов был гибким и не зависел от конкретных классов?

#### ❌ Без паттерна (жёсткая привязка)

```python
class OrderProcessor:
    def create_delivery(self, delivery_type: str):
        """Проблема: при добавлении нового типа доставки нужно менять код."""
        if delivery_type == "courier":
            return CourierDelivery()
        elif delivery_type == "pickup":
            return PickupDelivery()
        elif delivery_type == "post":
            return PostDelivery()
        # Каждый раз добавляем elif...
2.2. Реализация Factory Method
Базовый пример
python
from abc import ABC, abstractmethod
from typing import Any, Dict


# ===== ПРОДУКТЫ =====
class Delivery(ABC):
    """Абстрактный продукт"""
    
    @abstractmethod
    def calculate_cost(self, weight: float, distance: float) -> float:
        pass
    
    @abstractmethod
    def deliver(self, address: str) -> str:
        pass


class CourierDelivery(Delivery):
    """Курьерская доставка"""
    
    def calculate_cost(self, weight: float, distance: float) -> float:
        return 5.0 * weight + 0.5 * distance
    
    def deliver(self, address: str) -> str:
        return f"Курьер доставит заказ по адресу {address}"


class PickupDelivery(Delivery):
    """Самовывоз"""
    
    def calculate_cost(self, weight: float, distance: float) -> float:
        return 0.0  # Бесплатно
    
    def deliver(self, address: str) -> str:
        return "Заказ ожидает в пункте выдачи"


class PostDelivery(Delivery):
    """Почтовая доставка"""
    
    def calculate_cost(self, weight: float, distance: float) -> float:
        return 2.0 * weight + 0.1 * distance
    
    def deliver(self, address: str) -> str:
        return f"Заказ отправлен почтой на {address}"


# ===== СОЗДАТЕЛИ (ФАБРИКИ) =====
class DeliveryFactory(ABC):
    """Абстрактный создатель"""
    
    @abstractmethod
    def create_delivery(self) -> Delivery:
        """Фабричный метод"""
        pass
    
    def process_order(self, weight: float, distance: float, address: str) -> Dict[str, Any]:
        """Общий метод, использующий фабричный метод"""
        delivery = self.create_delivery()
        cost = delivery.calculate_cost(weight, distance)
        info = delivery.deliver(address)
        
        return {
            "delivery_type": delivery.__class__.__name__,
            "cost": cost,
            "info": info
        }


class CourierDeliveryFactory(DeliveryFactory):
    def create_delivery(self) -> Delivery:
        return CourierDelivery()


class PickupDeliveryFactory(DeliveryFactory):
    def create_delivery(self) -> Delivery:
        return PickupDelivery()


class PostDeliveryFactory(DeliveryFactory):
    def create_delivery(self) -> Delivery:
        return PostDelivery()


# ===== ИСПОЛЬЗОВАНИЕ =====
def main():
    weight = 2.5  # кг
    distance = 10  # км
    address = "г. Москва, ул. Тверская, д. 1"
    
    factories = [
        CourierDeliveryFactory(),
        PickupDeliveryFactory(),
        PostDeliveryFactory()
    ]
    
    for factory in factories:
        result = factory.process_order(weight, distance, address)
        print(f"Доставка: {result['delivery_type']}")
        print(f"  Стоимость: {result['cost']:.2f} руб.")
        print(f"  {result['info']}\n")


if __name__ == "__main__":
    main()
Вывод:

text
Доставка: CourierDelivery
  Стоимость: 17.50 руб.
  Курьер доставит заказ по адресу г. Москва, ул. Тверская, д. 1

Доставка: PickupDelivery
  Стоимость: 0.00 руб.
  Заказ ожидает в пункте выдачи

Доставка: PostDelivery
  Стоимость: 6.00 руб.
  Заказ отправлен почтой на г. Москва, ул. Тверская, д. 1
2.3. Factory Method с параметрами
python
from abc import ABC, abstractmethod
from enum import Enum
from typing import Type


class PaymentMethod(Enum):
    """Типы платёжных методов"""
    CARD = "card"
    PAYPAL = "paypal"
    CRYPTO = "crypto"


class Payment(ABC):
    """Абстрактный платёжный метод"""
    
    @abstractmethod
    def pay(self, amount: float) -> bool:
        pass


class CardPayment(Payment):
    def __init__(self, card_number: str, cvv: str):
        self.card_number = card_number
        self.cvv = cvv
    
    def pay(self, amount: float) -> bool:
        print(f"Оплата картой {self.card_number[-4:]} на сумму {amount} руб.")
        return True


class PayPalPayment(Payment):
    def __init__(self, email: str):
        self.email = email
    
    def pay(self, amount: float) -> bool:
        print(f"Оплата через PayPal ({self.email}) на сумму {amount} руб.")
        return True


class CryptoPayment(Payment):
    def __init__(self, wallet_address: str):
        self.wallet_address = wallet_address
    
    def pay(self, amount: float) -> bool:
        print(f"Оплата криптовалютой с кошелька {self.wallet_address[:10]}... на сумму {amount} руб.")
        return True


class PaymentFactory:
    """Фабрика с методом, принимающим параметры"""
    
    @staticmethod
    def create_payment(method: PaymentMethod, **kwargs) -> Payment:
        """
        Фабричный метод с параметрами.
        
        Args:
            method: Тип платежа
            **kwargs: Параметры для конкретного платёжного метода
        
        Returns:
            Объект Payment
        """
        if method == PaymentMethod.CARD:
            return CardPayment(kwargs['card_number'], kwargs['cvv'])
        elif method == PaymentMethod.PAYPAL:
            return PayPalPayment(kwargs['email'])
        elif method == PaymentMethod.CRYPTO:
            return CryptoPayment(kwargs['wallet_address'])
        else:
            raise ValueError(f"Unknown payment method: {method}")


# Использование
payment = PaymentFactory.create_payment(
    PaymentMethod.CARD,
    card_number="1234-5678-9012-3456",
    cvv="123"
)
payment.pay(1000)
2.4. Когда использовать Factory Method
Ситуация	Применение
Код не знает, объекты каких классов ему нужно создать	✅ Да
Подклассы должны определять, какие объекты создавать	✅ Да
Класс делегирует создание объектов наследникам	✅ Да
Простое создание одного типа объектов	❌ Нет (проще __init__)
Нужно создать семейство связанных объектов	❌ Нет (использовать Abstract Factory)
Часть 3. Паттерн Singleton (Одиночка)
3.1. Назначение и проблема
Назначение: Гарантирует, что класс имеет только один экземпляр, и предоставляет глобальную точку доступа к нему.

Проблема: Как обеспечить, чтобы у класса был только один объект?

Типичные примеры использования:

Логгер (один лог-файл)

Конфигурация приложения

Пул соединений с БД

Кэш

Менеджер ресурсов

3.2. Реализация Singleton в Python
Способ 1: Через __new__ (классический)
python
class Logger:
    """Одиночка — логгер приложения"""
    
    _instance = None
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            print("Создание первого и единственного экземпляра Logger")
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self, log_file: str = "app.log"):
        # Инициализация происходит при каждом вызове!
        # Но мы можем проверить, инициализирован ли уже объект
        if not hasattr(self, 'initialized'):
            print(f"Инициализация логгера с файлом {log_file}")
            self.log_file = log_file
            self.initialized = True
    
    def log(self, message: str):
        with open(self.log_file, "a", encoding='utf-8') as f:
            f.write(f"{message}\n")
        print(f"[LOG] {message}")


# Проверка
logger1 = Logger()
logger2 = Logger()
logger3 = Logger()

print(f"logger1 is logger2: {logger1 is logger2}")  # True
print(f"logger1 is logger3: {logger1 is logger3}")  # True

logger1.log("Сообщение 1")
logger2.log("Сообщение 2")
Недостаток: __init__ вызывается при каждом создании "объекта".

Способ 2: С защитой повторной инициализации
python
class Config:
    """Одиночка для хранения конфигурации"""
    
    _instance = None
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        # Инициализируем только один раз
        if not hasattr(self, '_initialized'):
            self._config = {}
            self._initialized = True
            print("Конфигурация инициализирована")
    
    def set(self, key: str, value):
        self._config[key] = value
    
    def get(self, key: str, default=None):
        return self._config.get(key, default)
    
    def load_from_file(self, filepath: str):
        import json
        with open(filepath, 'r', encoding='utf-8') as f:
            self._config = json.load(f)
    
    def save_to_file(self, filepath: str):
        import json
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(self._config, f, indent=2, ensure_ascii=False)


# Использование
config1 = Config()
config2 = Config()

config1.set("app_name", "MyApp")
config1.set("version", "1.0.0")

print(config2.get("app_name"))  # "MyApp"
print(config1 is config2)  # True
Способ 3: Через декоратор
python
def singleton(cls):
    """Декоратор для превращения класса в одиночку"""
    instances = {}
    
    def get_instance(*args, **kwargs):
        if cls not in instances:
            print(f"Создание экземпляра {cls.__name__}")
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance


@singleton
class DatabaseConnection:
    def __init__(self, host: str = "localhost"):
        print(f"Подключение к БД на {host}")
        self.host = host
        self.connected = True
    
    def query(self, sql: str):
        print(f"Выполнение запроса на {self.host}: {sql}")
        return []


# Проверка
db1 = DatabaseConnection("localhost")
db2 = DatabaseConnection("remote_host")  # Параметр игнорируется!
db3 = DatabaseConnection()

print(f"db1 is db2: {db1 is db2}")  # True
print(f"db1.host: {db1.host}")  # localhost
Внимание: При таком подходе параметры при повторном вызове игнорируются.

Способ 4: Через метакласс (самый правильный в Python)
python
class SingletonMeta(type):
    """Метакласс для реализации Singleton"""
    
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            print(f"Создание экземпляра {cls.__name__}")
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]


class AppSettings(metaclass=SingletonMeta):
    """Настройки приложения — Singleton через метакласс"""
    
    def __init__(self):
        # Инициализация будет выполнена только один раз!
        print("Инициализация настроек")
        self._settings = {
            "theme": "dark",
            "language": "ru"
        }
    
    def get(self, key: str):
        return self._settings.get(key)
    
    def set(self, key: str, value):
        self._settings[key] = value


# Проверка
settings1 = AppSettings()
settings2 = AppSettings()

settings1.set("theme", "light")
print(settings2.get("theme"))  # "light"
print(settings1 is settings2)  # True
Способ 5: Модульный Singleton (Pythonic way)
В Python самый простой и питоничный способ — использовать модуль:

python
# config_module.py
class _Config:
    def __init__(self):
        self._settings = {}
    
    def set(self, key, value):
        self._settings[key] = value
    
    def get(self, key, default=None):
        return self._settings.get(key, default)


# Создаём единственный экземпляр на уровне модуля
config = _Config()


# app.py — использование
from config_module import config

config.set("app_name", "MyApp")
print(config.get("app_name"))
Такой подход:

Не требует специальной реализации

Работает за счёт особенностей импорта в Python

Является самым рекомендуемым

3.3. Проблемы и антипаттерны Singleton
Проблема	Описание	Решение
Глобальное состояние	Усложняет отладку и тестирование	Использовать Dependency Injection
Скрытые зависимости	Код неявно зависит от синглтона	Явно передавать зависимости
Проблемы с многопоточностью	Небезопасен в потоках	Добавить блокировки
Сложность тестирования	Нельзя создать новый экземпляр для теста	Использовать фабрики
Многопоточная версия Singleton
python
import threading


class ThreadSafeSingleton:
    """Потокобезопасный Singleton"""
    
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance


# Или с использованием декоратора
def thread_safe_singleton(cls):
    instances = {}
    lock = threading.Lock()
    
    def get_instance(*args, **kwargs):
        if cls not in instances:
            with lock:
                if cls not in instances:
                    instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance
Часть 4. Сравнение паттернов
4.1. Factory Method vs Singleton
Характеристика	Factory Method	Singleton
Цель	Создание объектов	Ограничение количества объектов
Количество объектов	Много	Один
Контроль создания	Делегируется подклассам	Скрыт внутри класса
Глобальный доступ	Нет (возвращается новый объект)	Да (один экземпляр)
Расширяемость	Легко (добавление новых фабрик)	Сложно (наследование нарушает)
4.2. Когда какой паттерн выбирать
python
# Пример: система оплаты заказов

# Singleton — для глобальных сервисов (один на всю систему)
class PaymentGateway(metaclass=SingletonMeta):
    """Единая платёжная система"""
    def process_payment(self, amount): ...

# Factory Method — для создания разных типов платежей
class PaymentFactory:
    def create_payment(self, method_type): ...

# Использование
gateway = PaymentGateway()  # Все используют одну платёжную систему
payment = PaymentFactory().create_payment("card")  # Конкретный платёж
gateway.process_payment(payment)
Часть 5. Практические примеры
5.1. Пример: Система логирования (Singleton)
python
import logging
from datetime import datetime
from enum import Enum
from pathlib import Path
from typing import Optional


class LogLevel(Enum):
    DEBUG = "DEBUG"
    INFO = "INFO"
    WARNING = "WARNING"
    ERROR = "ERROR"


class Logger(metaclass=SingletonMeta):
    """Единый логгер для всего приложения"""
    
    def __init__(self, log_dir: str = "logs", max_size_mb: int = 10):
        if not hasattr(self, '_initialized'):
            self.log_dir = Path(log_dir)
            self.log_dir.mkdir(exist_ok=True)
            self.max_size_mb = max_size_mb
            self._current_log = self._get_current_log_file()
            self._initialized = True
    
    def _get_current_log_file(self) -> Path:
        """Возвращает файл лога для текущего дня"""
        today = datetime.now().strftime("%Y-%m-%d")
        return self.log_dir / f"{today}.log"
    
    def _check_rotation(self):
        """Проверяет, не превышен ли размер файла"""
        if self._current_log.exists():
            size_mb = self._current_log.stat().st_size / (1024 * 1024)
            if size_mb >= self.max_size_mb:
                timestamp = datetime.now().strftime("%H%M%S")
                new_name = self.log_dir / f"{self._current_log.stem}_{timestamp}.log"
                self._current_log.rename(new_name)
                self._current_log = self._get_current_log_file()
    
    def log(self, level: LogLevel, message: str):
        """Записывает сообщение в лог"""
        self._check_rotation()
        timestamp = datetime.now().isoformat()
        log_line = f"[{timestamp}] {level.value}: {message}\n"
        
        with open(self._current_log, "a", encoding='utf-8') as f:
            f.write(log_line)
        
        # Также выводим в консоль для DEBUG
        if level == LogLevel.DEBUG:
            print(log_line.strip())
    
    def debug(self, message: str):
        self.log(LogLevel.DEBUG, message)
    
    def info(self, message: str):
        self.log(LogLevel.INFO, message)
    
    def warning(self, message: str):
        self.log(LogLevel.WARNING, message)
    
    def error(self, message: str):
        self.log(LogLevel.ERROR, message)


# Использование в любом модуле
class UserService:
    def register_user(self, email: str):
        logger = Logger()  # Всегда один и тот же экземпляр
        logger.info(f"Регистрация пользователя: {email}")
        # ... логика регистрации
        logger.debug(f"Пользователь {email} зарегистрирован")
5.2. Пример: Фабрика уведомлений
python
from abc import ABC, abstractmethod
from typing import Dict, Any


class Notification(ABC):
    """Абстрактное уведомление"""
    
    @abstractmethod
    def send(self, recipient: str, message: str) -> bool:
        pass


class EmailNotification(Notification):
    def send(self, recipient: str, message: str) -> bool:
        print(f"Отправка EMAIL на {recipient}: {message}")
        return True


class SMSNotification(Notification):
    def send(self, recipient: str, message: str) -> bool:
        # Ограничение длины SMS
        if len(message) > 160:
            message = message[:157] + "..."
        print(f"Отправка SMS на {recipient}: {message}")
        return True


class PushNotification(Notification):
    def send(self, recipient: str, message: str) -> bool:
        print(f"Отправка PUSH уведомления {recipient}: {message}")
        return True


class TelegramNotification(Notification):
    def send(self, recipient: str, message: str) -> bool:
        print(f"Отправка TELEGRAM @{recipient}: {message}")
        return True


class NotificationFactory:
    """Фабрика для создания уведомлений"""
    
    _creators = {
        "email": EmailNotification,
        "sms": SMSNotification,
        "push": PushNotification,
        "telegram": TelegramNotification,
    }
    
    @classmethod
    def create(cls, notification_type: str) -> Notification:
        """Создаёт уведомление указанного типа"""
        creator = cls._creators.get(notification_type.lower())
        if not creator:
            raise ValueError(f"Unknown notification type: {notification_type}")
        return creator()
    
    @classmethod
    def register_type(cls, name: str, notification_class):
        """Регистрация нового типа уведомлений (расширение без изменения кода)"""
        cls._creators[name.lower()] = notification_class


# Использование
class NotificationService:
    def __init__(self, factory: NotificationFactory = None):
        self.factory = factory or NotificationFactory()
    
    def send(self, notification_type: str, recipient: str, message: str):
        notification = self.factory.create(notification_type)
        return notification.send(recipient, message)


# Демонстрация
service = NotificationService()
service.send("email", "user@example.com", "Добро пожаловать!")
service.send("sms", "+71234567890", "Ваш код: 123456")
service.send("telegram", "username", "Новое сообщение")
Часть 6. Стиль и синтаксис
6.1. Рекомендации по стилю
python
# ===== ХОРОШИЙ СТИЛЬ =====

# 1. Имена классов — PascalCase
class MySingleton: pass
class PaymentFactory: pass

# 2. Имена методов и атрибутов — snake_case
def create_payment(): pass
def get_instance(): pass

# 3. Приватные атрибуты с одним подчёркиванием
self._instance = None
self._initialized = False

# 4. Защищённые методы с одним подчёркиванием
def _ensure_instance(self): pass

# 5. Использование аннотаций типов
def create(cls, config: dict) -> 'Product': pass

# 6. Документирование
class Singleton:
    """
    Реализация паттерна Singleton в Python.
    
    Использование:
        s1 = Singleton()
        s2 = Singleton()
        assert s1 is s2
    """
    pass


# ===== ЧЕГО СЛЕДУЕТ ИЗБЕГАТЬ =====

# Плохо: магические методы без необходимости
class Bad:
    def __new__(cls):
        # Непонятно, зачем
        pass

# Плохо: двойное подчёркивание для приватных (без необходимости)
class Bad:
    def __secret_method(self):  # Излишнее искажение имени
        pass

# Плохо: пустой __init__ в Singleton
class Bad:
    def __init__(self):
        pass  # Бесполезно, если нет логики
6.2. Шаблоны реализации
python
# === ШАБЛОН FACTORY METHOD ===

from abc import ABC, abstractmethod

class Product(ABC):
    @abstractmethod
    def operation(self): pass

class ConcreteProductA(Product):
    def operation(self): return "Result A"

class ConcreteProductB(Product):
    def operation(self): return "Result B"

class Creator(ABC):
    @abstractmethod
    def factory_method(self) -> Product:
        pass
    
    def some_operation(self):
        product = self.factory_method()
        return product.operation()

class ConcreteCreatorA(Creator):
    def factory_method(self) -> Product:
        return ConcreteProductA()

class ConcreteCreatorB(Creator):
    def factory_method(self) -> Product:
        return ConcreteProductB()


# === ШАБЛОН SINGLETON (метакласс) ===

class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Singleton(metaclass=SingletonMeta):
    """Класс-одиночка"""
    pass


# === ШАБЛОН SINGLETON (модульный) ===

# module.py
class _MySingleton:
    def method(self): pass

instance = _MySingleton()

# другой_модуль.py
from module import instance
Контрольные вопросы
Что такое паттерн Factory Method? Какую проблему он решает?

В чём разница между Factory Method и просто вызовом конструктора?

Когда нужно использовать паттерн Singleton? Приведите 3 примера.

Какие проблемы создаёт использование Singleton?

Почему в Python модульный Singleton считается наиболее правильным?

Как сделать Singleton потокобезопасным?

Как расширить Factory Method без изменения существующего кода?

Можно ли комбинировать Factory Method и Singleton? Приведите пример.

Практическое задание
Задание 1 (базовое)
Реализуйте паттерн Singleton для класса Configuration, который загружает настройки из JSON-файла и предоставляет доступ к ним во всём приложении.

Задание 2 (среднее)
Реализуйте Factory Method для создания различных типов документов (PDF, DOCX, TXT, HTML) с методами save() и open().

Задание 3 (сложное)
Разработайте систему кэширования, где:

CacheFactory создаёт различные стратегии кэширования (LRU, FIFO, TTL)

CacheManager — Singleton, управляющий экземплярами кэша

Возможность регистрации новых стратегий без изменения кода

Шпаргалка
python
# === FACTORY METHOD ===

# Когда использовать: создание объектов разных типов с общим интерфейсом
# Плюсы: расширяемость, отделение логики создания
# Минусы: много маленьких классов

class Product(ABC):
    @abstractmethod
    def do(self): pass

class Creator(ABC):
    @abstractmethod
    def factory(self) -> Product: pass
    
    def use(self):
        return self.factory().do()


# === SINGLETON ===

# Когда использовать: глобальные сервисы (логгер, конфиг, пул соединений)
# Плюсы: один экземпляр, глобальный доступ
# Минусы: глобальное состояние, проблемы с тестированием

# Способ 1: метакласс (рекомендуется для сложных случаев)
class SingletonMeta(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

# Способ 2: модуль (рекомендуется для простых случаев)
# singleton.py
instance = MyClass()

# Способ 3: декоратор
def singleton(cls):
    instances = {}
    def get(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get
Итог лекции
Вы сегодня:

Изучили порождающие паттерны Factory Method и Singleton

Узнали проблемы, которые они решают

Освоили различные способы реализации в Python

Научились выбирать подходящий паттерн для конкретной задачи

Познакомились со стилем и синтаксисом реализации

Теперь вы можете проектировать гибкие, расширяемые и поддерживаемые системы, используя проверенные паттерны!

