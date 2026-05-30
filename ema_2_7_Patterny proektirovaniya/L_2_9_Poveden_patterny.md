# Тема 2.9. Поведенческие паттерны: Strategy, Observer. Структурные: Adapter. Понятие спецификации языка программирования

**Цель лекции:**  
Изучить поведенческие паттерны Strategy и Observer, структурный паттерн Adapter, понять их применение в Python, а также познакомиться с понятием спецификации языка программирования и его значением для разработчика.

> Главная мысль: **Паттерны проектирования — это язык общения между разработчиками. Название паттерна мгновенно передаёт понимание решения коллегам.**

---

## Часть 1. Понятие спецификации языка программирования

### 1.1. Что такое спецификация языка

**Спецификация языка программирования** — это официальный документ, который точно и полно описывает синтаксис, семантику и поведение языка.

**Для чего нужна спецификация:**
┌─────────────────────────────────────────────────────────────┐
│ РОЛЬ СПЕЦИФИКАЦИИ │
├─────────────────────────────────────────────────────────────┤
│ 1. Единое понимание языка (все разработчики читают одно) │
│ 2. Основа для реализации компиляторов и интерпретаторов │
│ 3. Гарантия совместимости между разными реализациями │
│ 4. Источник истины при спорных вопросах │
│ 5. Документация для разработчиков │
└─────────────────────────────────────────────────────────────┘

text

### 1.2. Спецификация Python

Python не имеет единого формального документа, но есть несколько авторитетных источников:

| Источник | Описание | Ссылка |
|----------|----------|--------|
| **PEP** | Python Enhancement Proposals | python.org/dev/peps |
| **PEP 8** | Стиль кода | Стандарт оформления |
| **PEP 20** | Zen of Python | Философия языка |
| **PEP 484** | Type Hints | Аннотации типов |
| **Python Docs** | Официальная документация | docs.python.org |
| **CPython** | Эталонная реализация | github.com/python/cpython |

### 1.3. Zen of Python (PEP 20)

```python
# Zen of Python — философия языка
import this

# Вывод:
"""
Beautiful is better than ugly.          # Красивое лучше уродливого
Explicit is better than implicit.       # Явное лучше неявного
Simple is better than complex.          # Простое лучше сложного
Complex is better than complicated.     # Сложное лучше запутанного
Flat is better than nested.             # Плоское лучше вложенного
Sparse is better than dense.            # Разреженное лучше плотного
Readability counts.                     # Читаемость имеет значение
Special cases aren't special enough.    # Особые случаи не настолько особые
Errors should never pass silently.      # Ошибки не должны замалчиваться
In the face of ambiguity, refuse the temptation to guess.  # При неясности откажись от догадок
There should be one obvious way to do it.  # Должен быть один очевидный способ
Now is better than never.               # Сейчас лучше, чем никогда
"""
1.4. Зачем разработчику знать спецификацию
Практическая польза:

python
# Пример: знание спецификации помогает понять поведение

# Вопрос: что выведет этот код?
def modify_list(lst):
    lst = [1, 2, 3]  # Создаётся новый объект
    return lst

my_list = [4, 5, 6]
result = modify_list(my_list)
print(my_list)  # [4, 5, 6] — оригинал не изменился!
print(result)   # [1, 2, 3]

# А это?
def modify_list_inplace(lst):
    lst.append(100)  # Изменяет оригинал

modify_list_inplace(my_list)
print(my_list)  # [4, 5, 6, 100]
Знание спецификации помогает:

Писать предсказуемый код

Понимать граничные случаи

Оптимизировать производительность

Общаться с другими разработчиками

Часть 2. Паттерн Strategy (Стратегия)
2.1. Назначение и проблема
Назначение: Определяет семейство алгоритмов, инкапсулирует каждый из них и делает их взаимозаменяемыми.

Проблема: Как сделать алгоритмы взаимозаменяемыми и не раздувать код условными операторами?

❌ Без паттерна (проблемный код)
python
class OrderCalculator:
    def calculate(self, order, discount_type):
        if discount_type == "percentage":
            return order.total * 0.9
        elif discount_type == "fixed":
            return order.total - 100
        elif discount_type == "seasonal":
            return order.total * 0.85
        elif discount_type == "clearance":
            return order.total * 0.7
        # Каждый раз добавляем elif → нарушение OCP
        return order.total
2.2. Реализация Strategy
python
from abc import ABC, abstractmethod
from typing import List, Dict, Any
from decimal import Decimal
from dataclasses import dataclass


# ===== СТРАТЕГИИ (алгоритмы) =====
class DiscountStrategy(ABC):
    """Абстрактный класс стратегии скидки"""
    
    @abstractmethod
    def calculate(self, total: Decimal) -> Decimal:
        pass


class PercentageDiscount(DiscountStrategy):
    """Процентная скидка"""
    
    def __init__(self, percent: Decimal):
        self.percent = percent
    
    def calculate(self, total: Decimal) -> Decimal:
        return total * (Decimal('1') - self.percent / Decimal('100'))


class FixedDiscount(DiscountStrategy):
    """Фиксированная скидка"""
    
    def __init__(self, amount: Decimal):
        self.amount = amount
    
    def calculate(self, total: Decimal) -> Decimal:
        return max(Decimal('0'), total - self.amount)


class SeasonalDiscount(DiscountStrategy):
    """Сезонная скидка"""
    
    def calculate(self, total: Decimal) -> Decimal:
        return total * Decimal('0.85')


class ClearanceDiscount(DiscountStrategy):
    """Уценённый товар (30% скидка)"""
    
    def calculate(self, total: Decimal) -> Decimal:
        return total * Decimal('0.7')


class LoyaltyDiscount(DiscountStrategy):
    """Скидка постоянного покупателя"""
    
    def __init__(self, years: int):
        self.years = years
    
    def calculate(self, total: Decimal) -> Decimal:
        discount = min(20, self.years * 2)
        return total * (Decimal('1') - Decimal(str(discount)) / Decimal('100'))


# ===== КЛИЕНТ =====
class Order:
    """Заказ, к которому применяется стратегия"""
    
    def __init__(self, items: List[Dict[str, Any]]):
        self.items = items
        self._total = None
    
    @property
    def total(self) -> Decimal:
        """Вычисляет общую сумму заказа"""
        if self._total is None:
            self._total = sum(
                Decimal(str(item['price'])) * item['quantity']
                for item in self.items
            )
        return self._total
    
    def apply_discount(self, strategy: DiscountStrategy) -> Decimal:
        """Применяет стратегию скидки"""
        return strategy.calculate(self.total)
    
    def __str__(self) -> str:
        return f"Order(total={self.total:.2f})"


class ShoppingCart:
    """Корзина с возможностью выбора стратегии скидки"""
    
    def __init__(self, discount_strategy: DiscountStrategy = None):
        self.items = []
        self._discount_strategy = discount_strategy
    
    def add_item(self, name: str, price: Decimal, quantity: int = 1):
        self.items.append({
            'name': name,
            'price': price,
            'quantity': quantity
        })
    
    def set_discount_strategy(self, strategy: DiscountStrategy):
        """Изменение стратегии во время выполнения"""
        self._discount_strategy = strategy
    
    def calculate_total(self) -> Decimal:
        order = Order(self.items)
        if self._discount_strategy:
            return order.apply_discount(self._discount_strategy)
        return order.total


# ===== ИСПОЛЬЗОВАНИЕ =====
def main():
    cart = ShoppingCart()
    
    # Добавляем товары
    cart.add_item("Ноутбук", Decimal('50000'), 1)
    cart.add_item("Мышь", Decimal('1500'), 2)
    cart.add_item("Клавиатура", Decimal('3000'), 1)
    
    print(f"Сумма без скидки: {cart.calculate_total():.2f} руб.")
    
    # Применяем разные стратегии
    strategies = [
        ("Процентная (10%)", PercentageDiscount(Decimal('10'))),
        ("Фиксированная (5000)", FixedDiscount(Decimal('5000'))),
        ("Сезонная", SeasonalDiscount()),
        ("Уценённые товары", ClearanceDiscount()),
        ("Постоянный клиент (5 лет)", LoyaltyDiscount(5)),
    ]
    
    for name, strategy in strategies:
        cart.set_discount_strategy(strategy)
        print(f"{name}: {cart.calculate_total():.2f} руб.")


if __name__ == "__main__":
    main()
Вывод:

text
Сумма без скидки: 54500.00 руб.
Процентная (10%): 49050.00 руб.
Фиксированная (5000): 49500.00 руб.
Сезонная: 46325.00 руб.
Уценённые товары: 38150.00 руб.
Постоянный клиент (5 лет): 49050.00 руб.
2.3. Strategy с использованием функций (Pythonic way)
python
from typing import Callable
from decimal import Decimal

# Стратегии как функции
def percentage_discount(percent: Decimal) -> Callable:
    def discount(total: Decimal) -> Decimal:
        return total * (1 - percent / 100)
    return discount

def fixed_discount(amount: Decimal) -> Callable:
    def discount(total: Decimal) -> Decimal:
        return max(Decimal('0'), total - amount)
    return discount

def loyalty_discount(years: int) -> Callable:
    def discount(total: Decimal) -> Decimal:
        discount_percent = min(20, years * 2)
        return total * (1 - discount_percent / 100)
    return discount


class FlexibleCart:
    def __init__(self, discount_func: Callable = None):
        self.items = []
        self.discount_func = discount_func
    
    def add_item(self, price: Decimal, quantity: int = 1):
        self.items.append((price, quantity))
    
    @property
    def total(self) -> Decimal:
        subtotal = sum(p * q for p, q in self.items)
        if self.discount_func:
            return self.discount_func(subtotal)
        return subtotal


# Использование
cart = FlexibleCart(percentage_discount(15))
cart.add_item(Decimal('1000'), 3)
print(f"Итого: {cart.total:.2f}")  # 2550.00
2.4. Когда использовать Strategy
Ситуация	Применение
Много вариантов алгоритма	✅ Да
Алгоритмы можно менять во время выполнения	✅ Да
Нужно избежать условных операторов	✅ Да
Алгоритм используется редко	❌ Нет (усложняет код)
Только один вариант	❌ Нет (достаточно функции)
Часть 3. Паттерн Observer (Наблюдатель)
3.1. Назначение и проблема
Назначение: Определяет механизм подписки, позволяющий одним объектам следить за изменениями других объектов.

Проблема: Как уведомить множество объектов об изменении состояния, не делая их жёстко связанными?

❌ Без паттерна (проблемный код)
python
class WeatherStation:
    def __init__(self):
        self.temperature = 0
    
    def set_temperature(self, temp):
        self.temperature = temp
        # Прямые вызовы → жёсткая связь
        display1.update(temp)
        display2.update(temp)
        logger.log(temp)
        mobile_app.notify(temp)
        # Каждый раз добавляем новый вызов...
3.2. Реализация Observer
python
from abc import ABC, abstractmethod
from typing import List, Set, Dict, Any
from dataclasses import dataclass
from datetime import datetime


# ===== ИНТЕРФЕЙСЫ =====
class Observer(ABC):
    """Интерфейс наблюдателя"""
    
    @abstractmethod
    def update(self, subject: 'Subject', event: str, data: Any) -> None:
        pass


class Subject(ABC):
    """Интерфейс наблюдаемого объекта"""
    
    @abstractmethod
    def attach(self, observer: Observer) -> None:
        pass
    
    @abstractmethod
    def detach(self, observer: Observer) -> None:
        pass
    
    @abstractmethod
    def notify(self, event: str, data: Any) -> None:
        pass


# ===== РЕАЛИЗАЦИЯ НАБЛЮДАЕМОГО ОБЪЕКТА =====
class WeatherStation(Subject):
    """Метеостанция — наблюдаемый объект"""
    
    def __init__(self):
        self._observers: Set[Observer] = set()
        self._temperature: float = 0.0
        self._humidity: float = 0.0
        self._pressure: float = 760.0
    
    def attach(self, observer: Observer) -> None:
        self._observers.add(observer)
        print(f"Подписан: {observer.__class__.__name__}")
    
    def detach(self, observer: Observer) -> None:
        self._observers.discard(observer)
        print(f"Отписан: {observer.__class__.__name__}")
    
    def notify(self, event: str, data: Any) -> None:
        for observer in self._observers:
            observer.update(self, event, data)
    
    # Бизнес-методы, которые вызывают уведомления
    def set_temperature(self, temperature: float) -> None:
        old_temp = self._temperature
        self._temperature = temperature
        self.notify("temperature_changed", {
            "old": old_temp,
            "new": temperature
        })
    
    def set_humidity(self, humidity: float) -> None:
        old_humidity = self._humidity
        self._humidity = humidity
        self.notify("humidity_changed", {
            "old": old_humidity,
            "new": humidity
        })
    
    def set_measurements(self, temp: float, humidity: float, pressure: float) -> None:
        self._temperature = temp
        self._humidity = humidity
        self._pressure = pressure
        self.notify("all_measurements", {
            "temperature": temp,
            "humidity": humidity,
            "pressure": pressure
        })
    
    @property
    def temperature(self) -> float:
        return self._temperature
    
    @property
    def humidity(self) -> float:
        return self._humidity
    
    @property
    def pressure(self) -> float:
        return self._pressure


# ===== НАБЛЮДАТЕЛИ =====
class CurrentConditionsDisplay(Observer):
    """Отображение текущих условий"""
    
    def update(self, subject: Subject, event: str, data: Any) -> None:
        if event == "all_measurements":
            print(f"[Дисплей] Текущие условия: {data['temperature']}°C, "
                  f"{data['humidity']}%, {data['pressure']} мм.рт.ст.")


class StatisticsDisplay(Observer):
    """Статистика наблюдений"""
    
    def __init__(self):
        self._temperatures: List[float] = []
    
    def update(self, subject: Subject, event: str, data: Any) -> None:
        if event == "temperature_changed":
            self._temperatures.append(data['new'])
            if self._temperatures:
                avg = sum(self._temperatures) / len(self._temperatures)
                print(f"[Статистика] Средняя температура: {avg:.1f}°C")


class AlertSystem(Observer):
    """Система оповещения о критических изменениях"""
    
    def __init__(self, temp_threshold: float = 35.0):
        self.temp_threshold = temp_threshold
    
    def update(self, subject: Subject, event: str, data: Any) -> None:
        if event == "temperature_changed" and data['new'] > self.temp_threshold:
            print(f"[ALERT] КРИТИЧЕСКАЯ ТЕМПЕРАТУРА: {data['new']}°C!")


class Logger(Observer):
    """Логирование всех событий"""
    
    def __init__(self, log_file: str = "weather.log"):
        self.log_file = log_file
    
    def update(self, subject: Subject, event: str, data: Any) -> None:
        timestamp = datetime.now().isoformat()
        with open(self.log_file, "a", encoding='utf-8') as f:
            f.write(f"{timestamp} | {event} | {data}\n")
        print(f"[Лог] Записано событие: {event}")


class MobileApp(Observer):
    """Мобильное приложение"""
    
    def __init__(self, user_id: str):
        self.user_id = user_id
    
    def update(self, subject: Subject, event: str, data: Any) -> None:
        if event == "all_measurements":
            print(f"[Mobile App] Пользователь {self.user_id}: "
                  f"температура {data['temperature']}°C")


# ===== ИСПОЛЬЗОВАНИЕ =====
def main():
    station = WeatherStation()
    
    # Создаём наблюдателей
    display = CurrentConditionsDisplay()
    statistics = StatisticsDisplay()
    alert = AlertSystem(temp_threshold=30.0)
    logger = Logger()
    mobile = MobileApp("user123")
    
    # Подписываем наблюдателей
    station.attach(display)
    station.attach(statistics)
    station.attach(alert)
    station.attach(logger)
    station.attach(mobile)
    
    print("\n--- Изменение температуры ---")
    station.set_temperature(25.0)
    
    print("\n--- Изменение всех параметров ---")
    station.set_measurements(28.5, 65.0, 755.0)
    
    print("\n--- Критическое изменение ---")
    station.set_temperature(32.0)
    
    print("\n--- Отписываем мобильное приложение ---")
    station.detach(mobile)
    
    print("\n--- Ещё одно изменение ---")
    station.set_temperature(29.0)


if __name__ == "__main__":
    main()
3.3. Встроенный Observer: событийная модель
python
from typing import Callable, Dict, List, Any


class Event:
    """Простая реализация событий"""
    
    def __init__(self):
        self._handlers: List[Callable] = []
    
    def subscribe(self, handler: Callable) -> None:
        """Подписка на событие"""
        self._handlers.append(handler)
    
    def unsubscribe(self, handler: Callable) -> None:
        """Отписка от события"""
        self._handlers.remove(handler)
    
    def emit(self, *args, **kwargs) -> None:
        """Вызов всех обработчиков"""
        for handler in self._handlers:
            handler(*args, **kwargs)


class Button:
    """Кнопка с событиями"""
    
    def __init__(self, label: str):
        self.label = label
        self.on_click = Event()
    
    def click(self):
        print(f"Нажата кнопка '{self.label}'")
        self.on_click.emit(self)


# Использование
def on_button_clicked(button: Button):
    print(f"Обработчик: кнопка '{button.label}' была нажата!")

def log_click(button: Button):
    print(f"[LOG] Нажатие кнопки {button.label} залогировано")

# Создаём кнопку
submit_button = Button("Submit")

# Подписываемся на событие
submit_button.on_click.subscribe(on_button_clicked)
submit_button.on_click.subscribe(log_click)

# Симулируем нажатие
submit_button.click()
3.4. Когда использовать Observer
Ситуация	Применение
Изменение состояния одного объекта должно влиять на другие	✅ Да
Количество наблюдателей неизвестно или меняется	✅ Да
Нужно слабое связывание между объектами	✅ Да
Простое прямое общение между двумя объектами	❌ Нет (избыточно)
Часть 4. Паттерн Adapter (Адаптер)
4.1. Назначение и проблема
Назначение: Преобразует интерфейс одного класса в интерфейс, ожидаемый клиентом.

Проблема: Как использовать класс с "неудобным" интерфейсом?

❌ Без паттерна (проблемный код)
python
# Сторонняя библиотека с неудобным интерфейсом
class LegacyPaymentSystem:
    def make_payment(self, amount_cents: int, card_data: dict) -> str:
        # Старая система работает с центомами и особым форматом
        return f"Payment of {amount_cents} cents processed"

# Наша система ожидает другой интерфейс
class NewPaymentProcessor:
    def pay(self, amount: float, card_number: str, cvv: str) -> bool:
        # Наш интерфейс: плавающая точка, отдельные поля
        pass
4.2. Реализация Adapter
python
from abc import ABC, abstractmethod
from typing import Dict, Any
from decimal import Decimal


# ===== ЦЕЛЕВОЙ ИНТЕРФЕЙС (ожидаемый клиентом) =====
class PaymentProcessor(ABC):
    """Интерфейс, который использует наша система"""
    
    @abstractmethod
    def process_payment(self, amount: Decimal, card_number: str, expiry: str, cvv: str) -> bool:
        pass
    
    @abstractmethod
    def refund_payment(self, transaction_id: str) -> bool:
        pass


# ===== АДАПТИРУЕМЫЕ КЛАССЫ (несовместимые интерфейсы) =====
class StripePayment:
    """Платёжная система Stripe"""
    
    def charge(self, amount_usd: float, token: str) -> str:
        print(f"Stripe: charging ${amount_usd} with token {token[:10]}...")
        return f"stripe_txn_{id(self)}"
    
    def reverse(self, charge_id: str) -> bool:
        print(f"Stripe: refunding {charge_id}")
        return True


class PayPalPayment:
    """Платёжная система PayPal"""
    
    def send_payment(self, amount: float, currency: str, email: str) -> str:
        print(f"PayPal: sending {amount} {currency} to {email}")
        return f"paypal_txn_{id(self)}"
    
    def refund(self, transaction_id: str) -> bool:
        print(f"PayPal: refunding {transaction_id}")
        return True


class LegacyPaymentSystem:
    """Старая платёжная система (работает с центами)"""
    
    def make_payment(self, amount_cents: int, card_data: Dict[str, str]) -> str:
        print(f"Legacy: payment of {amount_cents} cents")
        return f"legacy_txn_{id(self)}"
    
    def cancel_payment(self, transaction_id: str) -> bool:
        print(f"Legacy: cancelling {transaction_id}")
        return True


# ===== АДАПТЕРЫ =====
class StripeAdapter(PaymentProcessor):
    """Адаптирует Stripe к нашему интерфейсу"""
    
    def __init__(self, stripe: StripePayment):
        self.stripe = stripe
    
    def process_payment(self, amount: Decimal, card_number: str, expiry: str, cvv: str) -> bool:
        # Преобразуем данные карты в токен (упрощённо)
        token = f"tok_{card_number[-4:]}_{expiry.replace('/', '')}"
        # Конвертируем рубли в доллары (упрощённо)
        amount_usd = float(amount) / 90.0
        # Вызываем Stripe API
        txn_id = self.stripe.charge(amount_usd, token)
        return bool(txn_id)
    
    def refund_payment(self, transaction_id: str) -> bool:
        return self.stripe.reverse(transaction_id)


class PayPalAdapter(PaymentProcessor):
    """Адаптирует PayPal к нашему интерфейсу"""
    
    def __init__(self, paypal: PayPalPayment, merchant_email: str):
        self.paypal = paypal
        self.merchant_email = merchant_email
    
    def process_payment(self, amount: Decimal, card_number: str, expiry: str, cvv: str) -> bool:
        # PayPal работает через email, а не через карту
        # Используем email пользователя (упрощённо)
        user_email = f"user_{card_number[-4:]}@example.com"
        amount_float = float(amount)
        txn_id = self.paypal.send_payment(amount_float, "RUB", user_email)
        return bool(txn_id)
    
    def refund_payment(self, transaction_id: str) -> bool:
        return self.paypal.refund(transaction_id)


class LegacyAdapter(PaymentProcessor):
    """Адаптирует Legacy систему к нашему интерфейсу"""
    
    def __init__(self, legacy: LegacyPaymentSystem):
        self.legacy = legacy
    
    def process_payment(self, amount: Decimal, card_number: str, expiry: str, cvv: str) -> bool:
        # Конвертируем рубли в копейки (центы)
        amount_cents = int(float(amount) * 100)
        # Подготавливаем данные карты в нужном формате
        card_data = {
            "card_number": card_number,
            "expiry": expiry,
            "cvv": cvv
        }
        txn_id = self.legacy.make_payment(amount_cents, card_data)
        return bool(txn_id)
    
    def refund_payment(self, transaction_id: str) -> bool:
        return self.legacy.cancel_payment(transaction_id)


# ===== КЛИЕНТСКИЙ КОД =====
class PaymentService:
    """Наш сервис работает только через интерфейс PaymentProcessor"""
    
    def __init__(self, processor: PaymentProcessor):
        self.processor = processor
    
    def pay(self, amount: Decimal, card_number: str, expiry: str, cvv: str) -> bool:
        print(f"\nОплата {amount} руб. через {self.processor.__class__.__name__}")
        return self.processor.process_payment(amount, card_number, expiry, cvv)
    
    def refund(self, transaction_id: str) -> bool:
        return self.processor.refund_payment(transaction_id)


# ===== ИСПОЛЬЗОВАНИЕ =====
def main():
    # Создаём адаптеры для разных платёжных систем
    stripe_adapter = StripeAdapter(StripePayment())
    paypal_adapter = PayPalAdapter(PayPalPayment(), "shop@example.com")
    legacy_adapter = LegacyAdapter(LegacyPaymentSystem())
    
    # Наш сервис может работать с любой платёжной системой через адаптер
    payment_data = {
        "amount": Decimal("1500.00"),
        "card_number": "1234-5678-9012-3456",
        "expiry": "12/25",
        "cvv": "123"
    }
    
    processors = [
        ("Stripe", stripe_adapter),
        ("PayPal", paypal_adapter),
        ("Legacy", legacy_adapter)
    ]
    
    for name, adapter in processors:
        service = PaymentService(adapter)
        result = service.pay(**payment_data)
        print(f"Результат: {'Успешно' if result else 'Ошибка'}\n")


if __name__ == "__main__":
    main()
4.3. Классический Adapter (через наследование)
python
# Адаптер через множественное наследование
class EuropeanPlug:
    """Европейская розетка (220V)"""
    def plug_in(self):
        return "220V"


class USPlug:
    """Американская розетка (110V)"""
    def connect(self):
        return "110V"


class EuropeanToUSAdapter(USPlug, EuropeanPlug):
    """Адаптер: европейская вилка в американскую розетку"""
    
    def connect(self):
        # Получаем 220V, преобразуем в 110V
        voltage = self.plug_in()
        return f"Converted {voltage} to 110V"


# Использование
def charge_laptop(plug: USPlug):
    print(f"Charging with {plug.connect()}")


european = EuropeanPlug()
adapter = EuropeanToUSAdapter()

# charge_laptop(european)  # Ошибка! Неправильный тип
charge_laptop(adapter)  # OK!
4.4. Когда использовать Adapter
Ситуация	Применение
Использование сторонней библиотеки с неудобным API	✅ Да
Интеграция со старым кодом (legacy)	✅ Да
Унификация разных интерфейсов	✅ Да
Простой вызов функции	❌ Нет (достаточно обёртки)
Часть 5. Сравнение паттернов
5.1. Strategy vs Observer vs Adapter
Характеристика	Strategy	Observer	Adapter
Категория	Поведенческий	Поведенческий	Структурный
Цель	Изменение алгоритма	Уведомление об изменениях	Приведение интерфейсов
Связь	Клиент → Стратегия	Субъект → Наблюдатели	Клиент → Адаптер → Сервис
Направление	Клиент выбирает	Субъект уведомляет	Преобразование
Изменяемость	Динамическая смена	Динамическая подписка	Фиксированная
5.2. Выбор паттерна
python
# ===== КОГДА ЧТО ВЫБРАТЬ =====

# Strategy — когда нужно менять алгоритм
class ImageProcessor:
    def __init__(self, filter_strategy):
        self.filter = filter_strategy
    
    def process(self, image):
        return self.filter.apply(image)

# Observer — когда нужно уведомлять о событиях
class StockPrice:
    def __init__(self):
        self.observers = []
    
    def set_price(self, price):
        self.price = price
        self.notify(price)

# Adapter — когда нужно интегрироваться с внешним API
class ExternalAPIAdapter:
    def get_user(self, user_id):
        external_data = self.legacy_api.fetch(user_id)
        return self._convert(external_data)
Контрольные вопросы
Что такое спецификация языка программирования и зачем она нужна?

В чём разница между паттернами Strategy и Observer?

Какую проблему решает паттерн Adapter? Приведите реальный пример.

Можно ли комбинировать несколько паттернов? Приведите пример.

В чём разница между адаптером и декоратором?

Как реализовать Observer без использования классов (только функции)?

Почему Strategy считается поведенческим паттерном?

Шпаргалка
python
# === STRATEGY ===
# Меняем алгоритм на лету
class Strategy(ABC):
    @abstractmethod
    def execute(self): pass

class Context:
    def __init__(self, strategy: Strategy):
        self._strategy = strategy
    
    def set_strategy(self, strategy: Strategy):
        self._strategy = strategy
    
    def do_something(self):
        return self._strategy.execute()


# === OBSERVER ===
# Уведомляем подписчиков об изменениях
class Observer(ABC):
    @abstractmethod
    def update(self, data): pass

class Subject:
    def __init__(self):
        self._observers = []
    
    def attach(self, observer): self._observers.append(observer)
    def notify(self, data):
        for obs in self._observers:
            obs.update(data)


# === ADAPTER ===
# Приводим несовместимые интерфейсы
class Target(ABC):
    @abstractmethod
    def request(self): pass

class Adaptee:
    def specific_request(self): return "specific"

class Adapter(Target):
    def __init__(self, adaptee: Adaptee):
        self._adaptee = adaptee
    
    def request(self):
        return self._adaptee.specific_request()
Практическое задание
Задание 1 (Strategy)
Реализуйте систему сортировки с разными стратегиями: быстрая сортировка, сортировка пузырьком, сортировка вставками. Пользователь должен выбирать стратегию во время выполнения.

Задание 2 (Observer)
Реализуйте систему оповещения о курсе валют. Подписчики (EmailNotifier, SMSNotifier, TelegramNotifier) получают уведомления при изменении курса более чем на 1%.

Задание 3 (Adapter)
Вам дана сторонняя библиотека для работы с изображениями с интерфейсом LegacyImageProcessor. Напишите адаптер, который позволит использовать её в вашей системе, ожидающей интерфейс ModernImageProcessor.

Итог лекции
Вы сегодня:

Познакомились с понятием спецификации языка программирования

Изучили поведенческие паттерны Strategy и Observer

Изучили структурный паттерн Adapter

Узнали, в каких ситуациях применять каждый паттерн

Научились реализовывать паттерны в Python

Теперь ваш арсенал пополнился тремя мощными паттернами, которые помогут писать гибкий, расширяемый и поддерживаемый код!
