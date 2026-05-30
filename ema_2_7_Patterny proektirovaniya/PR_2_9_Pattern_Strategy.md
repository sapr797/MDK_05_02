# ПЗ 2.9. Реализация паттерна Strategy

**Тема:** Поведенческие паттерны (Strategy)

**Цель работы:**  
Научиться реализовывать паттерн Strategy на Python, понимать, в каких ситуациях его применять, уметь проектировать гибкие системы с взаимозаменяемыми алгоритмами.

**Время выполнения:** 90-120 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `pylint`, `radon`

```bash
pip install pylint radon
Нулевой вариант (эталонный) — Демонстрация преподавателя
Назначение: преподаватель выполняет этот вариант на занятии, показывая реализацию паттерна Strategy.

Техническое задание (нулевой вариант)
Разработайте систему для расчёта стоимости доставки заказов с использованием паттерна Strategy. Система должна поддерживать различные стратегии доставки: курьерская, почтовая, самовывоз, экспресс-доставка. Стратегии должны быть взаимозаменяемыми во время выполнения.

Эталонная реализация
python
#!/usr/bin/env python3
"""
Модуль для расчёта стоимости доставки с использованием паттерна Strategy.
"""

from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import List, Dict, Any, Optional, Callable
from decimal import Decimal
from enum import Enum
from datetime import datetime, timedelta


# ============================================================
# МОДЕЛИ ДАННЫХ
# ============================================================

@dataclass
class Address:
    """Адрес доставки"""
    city: str
    street: str
    house: str
    apartment: Optional[str] = None
    postal_code: Optional[str] = None
    
    def __str__(self) -> str:
        return f"{self.city}, {self.street}, {self.house}"


@dataclass
class OrderItem:
    """Товар в заказе"""
    name: str
    weight_kg: float
    price: Decimal
    quantity: int
    
    @property
    def total_weight(self) -> float:
        return self.weight_kg * self.quantity
    
    @property
    def total_price(self) -> Decimal:
        return self.price * self.quantity


@dataclass
class Order:
    """Заказ"""
    items: List[OrderItem]
    delivery_address: Address
    created_at: datetime = None
    
    def __post_init__(self):
        if self.created_at is None:
            self.created_at = datetime.now()
    
    @property
    def total_weight(self) -> float:
        return sum(item.total_weight for item in self.items)
    
    @property
    def total_price(self) -> Decimal:
        return sum(item.total_price for item in self.items, Decimal('0'))


# ============================================================
    # СТРАТЕГИИ ДОСТАВКИ (РЕАЛИЗАЦИЯ ПАТТЕРНА)
# ============================================================

class DeliveryStrategy(ABC):
    """Абстрактная стратегия доставки"""
    
    @abstractmethod
    def calculate_cost(self, order: Order) -> Decimal:
        """Расчёт стоимости доставки"""
        pass
    
    @abstractmethod
    def estimate_delivery_time(self, order: Order) -> timedelta:
        """Оценка времени доставки"""
        pass
    
    @abstractmethod
    def get_description(self) -> str:
        """Описание стратегии"""
        pass


class CourierDelivery(DeliveryStrategy):
    """Стратегия: курьерская доставка"""
    
    def __init__(self, base_rate: Decimal = Decimal('200'), per_kg_rate: Decimal = Decimal('50')):
        self.base_rate = base_rate
        self.per_kg_rate = per_kg_rate
    
    def calculate_cost(self, order: Order) -> Decimal:
        # Базовая стоимость + стоимость за кг
        weight_cost = Decimal(str(order.total_weight)) * self.per_kg_rate
        return self.base_rate + weight_cost
    
    def estimate_delivery_time(self, order: Order) -> timedelta:
        # Курьерская доставка: 1-2 дня
        return timedelta(days=1, hours=12)
    
    def get_description(self) -> str:
        return f"Курьерская доставка (база: {self.base_rate}₽ + {self.per_kg_rate}₽/кг)"


class PostDelivery(DeliveryStrategy):
    """Стратегия: почтовая доставка"""
    
    def __init__(self, base_rate: Decimal = Decimal('100'), per_kg_rate: Decimal = Decimal('30')):
        self.base_rate = base_rate
        self.per_kg_rate = per_kg_rate
    
    def calculate_cost(self, order: Order) -> Decimal:
        weight_cost = Decimal(str(order.total_weight)) * self.per_kg_rate
        return self.base_rate + weight_cost
    
    def estimate_delivery_time(self, order: Order) -> timedelta:
        # Почта: 3-7 дней
        return timedelta(days=5)
    
    def get_description(self) -> str:
        return f"Почтовая доставка (база: {self.base_rate}₽ + {self.per_kg_rate}₽/кг)"


class PickupDelivery(DeliveryStrategy):
    """Стратегия: самовывоз (бесплатно)"""
    
    def calculate_cost(self, order: Order) -> Decimal:
        return Decimal('0')
    
    def estimate_delivery_time(self, order: Order) -> timedelta:
        # Самовывоз: сразу после оформления
        return timedelta(hours=1)
    
    def get_description(self) -> str:
        return "Самовывоз (бесплатно)"


class ExpressDelivery(DeliveryStrategy):
    """Стратегия: экспресс-доставка"""
    
    def __init__(self, base_rate: Decimal = Decimal('500'), per_kg_rate: Decimal = Decimal('100')):
        self.base_rate = base_rate
        self.per_kg_rate = per_kg_rate
    
    def calculate_cost(self, order: Order) -> Decimal:
        weight_cost = Decimal(str(order.total_weight)) * self.per_kg_rate
        # Срочная доставка имеет повышенный коэффициент
        return (self.base_rate + weight_cost) * Decimal('1.5')
    
    def estimate_delivery_time(self, order: Order) -> timedelta:
        # Экспресс: 2-4 часа
        return timedelta(hours=3)
    
    def get_description(self) -> str:
        return f"Экспресс-доставка (база: {self.base_rate}₽ + {self.per_kg_rate}₽/кг × 1.5)"


class InternationalDelivery(DeliveryStrategy):
    """Стратегия: международная доставка"""
    
    def __init__(self, country: str, base_rate: Decimal = Decimal('1500'), per_kg_rate: Decimal = Decimal('200')):
        self.country = country
        self.base_rate = base_rate
        self.per_kg_rate = per_kg_rate
    
    def calculate_cost(self, order: Order) -> Decimal:
        weight_cost = Decimal(str(order.total_weight)) * self.per_kg_rate
        return self.base_rate + weight_cost
    
    def estimate_delivery_time(self, order: Order) -> timedelta:
        # Международная: 7-14 дней
        return timedelta(days=10)
    
    def get_description(self) -> str:
        return f"Международная доставка в {self.country} (база: {self.base_rate}₽ + {self.per_kg_rate}₽/кг)"


# ============================================================
    # ДОПОЛНИТЕЛЬНЫЕ СТРАТЕГИИ (С КОМИССИЕЙ)
# ============================================================

class PremiumDelivery(DeliveryStrategy):
    """Премиум-доставка (страховка + приоритет)"""
    
    def __init__(self, base_strategy: DeliveryStrategy, insurance_percent: float = 0.05):
        self.base_strategy = base_strategy
        self.insurance_percent = insurance_percent
    
    def calculate_cost(self, order: Order) -> Decimal:
        base_cost = self.base_strategy.calculate_cost(order)
        insurance = base_cost * Decimal(str(self.insurance_percent))
        return base_cost + insurance
    
    def estimate_delivery_time(self, order: Order) -> timedelta:
        # Премиум-доставка быстрее на 20%
        base_time = self.base_strategy.estimate_delivery_time(order)
        return base_time * 0.8
    
    def get_description(self) -> str:
        return f"Премиум: {self.base_strategy.get_description()} + страховка {self.insurance_percent*100}%"


# ============================================================
    # ФАБРИКА СТРАТЕГИЙ (ДЛЯ УДОБНОГО СОЗДАНИЯ)
# ============================================================

class DeliveryStrategyFactory:
    """Фабрика для создания стратегий доставки"""
    
    _strategies = {
        "courier": CourierDelivery,
        "post": PostDelivery,
        "pickup": PickupDelivery,
        "express": ExpressDelivery,
    }
    
    @classmethod
    def create(cls, strategy_type: str, **kwargs) -> DeliveryStrategy:
        """Создаёт стратегию по типу"""
        strategy_class = cls._strategies.get(strategy_type.lower())
        if not strategy_class:
            raise ValueError(f"Unknown strategy type: {strategy_type}")
        return strategy_class(**kwargs)
    
    @classmethod
    def register(cls, name: str, strategy_class):
        """Регистрация новой стратегии"""
        cls._strategies[name.lower()] = strategy_class
    
    @classmethod
    def get_available(cls) -> List[str]:
        """Список доступных стратегий"""
        return list(cls._strategies.keys())


# ============================================================
    # КЛИЕНТСКИЙ КОД
# ============================================================

class OrderProcessor:
    """Обработчик заказов, использующий стратегию доставки"""
    
    def __init__(self, delivery_strategy: Optional[DeliveryStrategy] = None):
        self._strategy = delivery_strategy
    
    def set_strategy(self, strategy: DeliveryStrategy) -> None:
        """Изменение стратегии во время выполнения"""
        self._strategy = strategy
    
    def get_strategy_description(self) -> str:
        """Получение описания текущей стратегии"""
        if self._strategy:
            return self._strategy.get_description()
        return "Стратегия не выбрана"
    
    def process_order(self, order: Order) -> Dict[str, Any]:
        """Обработка заказа с текущей стратегией"""
        if not self._strategy:
            raise ValueError("Delivery strategy not set")
        
        return {
            "order": order,
            "delivery_cost": self._strategy.calculate_cost(order),
            "delivery_time": self._strategy.estimate_delivery_time(order),
            "strategy": self._strategy.get_description(),
            "total": order.total_price + self._strategy.calculate_cost(order)
        }


# ============================================================
    # ФУНКЦИОНАЛЬНЫЙ ПОДХОД (Pythonic way)
# ============================================================

# Стратегии как простые функции
def calculate_courier_cost(order: Order, base_rate: float = 200, per_kg_rate: float = 50) -> float:
    return base_rate + order.total_weight * per_kg_rate

def calculate_post_cost(order: Order, base_rate: float = 100, per_kg_rate: float = 30) -> float:
    return base_rate + order.total_weight * per_kg_rate

def calculate_pickup_cost(order: Order) -> float:
    return 0.0

def calculate_express_cost(order: Order, base_rate: float = 500, per_kg_rate: float = 100) -> float:
    return (base_rate + order.total_weight * per_kg_rate) * 1.5


class FunctionalOrderProcessor:
    """Обработчик заказов с функциональными стратегиями"""
    
    def __init__(self, cost_calculator: Callable[[Order], float]):
        self._cost_calculator = cost_calculator
    
    def set_calculator(self, calculator: Callable[[Order], float]) -> None:
        self._cost_calculator = calculator
    
    def calculate_delivery_cost(self, order: Order) -> float:
        return self._cost_calculator(order)


# ============================================================
    # ДЕМОНСТРАЦИЯ
# ============================================================

def create_sample_order() -> Order:
    """Создание тестового заказа"""
    items = [
        OrderItem("Ноутбук", 2.5, Decimal('50000'), 1),
        OrderItem("Мышь", 0.2, Decimal('1500'), 2),
        OrderItem("Клавиатура", 0.8, Decimal('3000'), 1),
    ]
    address = Address("Москва", "Тверская", "15", "42")
    return Order(items, address)


def demo_classic_strategy():
    """Демонстрация классической реализации Strategy"""
    print("\n" + "=" * 70)
    print("ЧАСТЬ 1: КЛАССИЧЕСКАЯ РЕАЛИЗАЦИЯ STRATEGY")
    print("=" * 70)
    
    order = create_sample_order()
    processor = OrderProcessor()
    
    print(f"\nЗаказ: {len(order.items)} товаров")
    print(f"Вес: {order.total_weight} кг")
    print(f"Стоимость товаров: {order.total_price:.2f} ₽")
    
    strategies = [
        CourierDelivery(),
        CourierDelivery(base_rate=Decimal('300'), per_kg_rate=Decimal('60')),
        PostDelivery(),
        PickupDelivery(),
        ExpressDelivery(),
        InternationalDelivery("Германия"),
        PremiumDelivery(CourierDelivery(), insurance_percent=0.1),
    ]
    
    print("\n" + "-" * 50)
    print("СРАВНЕНИЕ СТРАТЕГИЙ ДОСТАВКИ")
    print("-" * 50)
    
    for strategy in strategies:
        processor.set_strategy(strategy)
        result = processor.process_order(order)
        print(f"\n{strategy.get_description()}")
        print(f"  Стоимость доставки: {result['delivery_cost']:.2f} ₽")
        print(f"  Время доставки: {result['delivery_time']}")
        print(f"  Итого: {result['total']:.2f} ₽")


def demo_strategy_switch():
    """Демонстрация смены стратегии во время выполнения"""
    print("\n" + "=" * 70)
    print("ЧАСТЬ 2: СМЕНА СТРАТЕГИИ ВО ВРЕМЯ ВЫПОЛНЕНИЯ")
    print("=" * 70)
    
    order = create_sample_order()
    processor = OrderProcessor()
    
    print(f"\nЗаказ на сумму: {order.total_price:.2f} ₽")
    
    # Пользователь выбирает способ доставки
    choices = [
        ("1", "Самовывоз", PickupDelivery()),
        ("2", "Почта", PostDelivery()),
        ("3", "Курьер", CourierDelivery()),
        ("4", "Экспресс", ExpressDelivery()),
    ]
    
    print("\nВыберите способ доставки:")
    for key, name, _ in choices:
        print(f"  {key}. {name}")
    
    # Симуляция выбора пользователя
    selected = "3"  # Для демонстрации выберем курьера
    
    for key, name, strategy in choices:
        if selected == key:
            processor.set_strategy(strategy)
            print(f"\nВы выбрали: {name}")
            result = processor.process_order(order)
            print(f"Стоимость доставки: {result['delivery_cost']:.2f} ₽")
            print(f"Время доставки: {result['delivery_time']}")
            print(f"Итого к оплате: {result['total']:.2f} ₽")
            break


def demo_factory_pattern():
    """Демонстрация использования фабрики стратегий"""
    print("\n" + "=" * 70)
    print("ЧАСТЬ 3: ФАБРИКА СТРАТЕГИЙ")
    print("=" * 70)
    
    order = create_sample_order()
    processor = OrderProcessor()
    
    print(f"\nДоступные стратегии: {DeliveryStrategyFactory.get_available()}")
    
    # Регистрация новой стратегии
    class WeekendDelivery(DeliveryStrategy):
        def calculate_cost(self, order: Order) -> Decimal:
            return Decimal('1000')
        
        def estimate_delivery_time(self, order: Order) -> timedelta:
            return timedelta(days=1)
        
        def get_description(self) -> str:
            return "Доставка в выходные дни"
    
    DeliveryStrategyFactory.register("weekend", WeekendDelivery)
    print(f"Добавлена стратегия 'weekend'")
    print(f"Доступные стратегии: {DeliveryStrategyFactory.get_available()}")
    
    # Создание стратегии через фабрику
    strategy = DeliveryStrategyFactory.create("express")
    processor.set_strategy(strategy)
    result = processor.process_order(order)
    
    print(f"\nСтратегия через фабрику: {strategy.get_description()}")
    print(f"Стоимость: {result['delivery_cost']:.2f} ₽")


def demo_functional_strategy():
    """Демонстрация функционального подхода"""
    print("\n" + "=" * 70)
    print("ЧАСТЬ 4: ФУНКЦИОНАЛЬНЫЙ ПОДХОД (PYTHONIC WAY)")
    print("=" * 70)
    
    order = create_sample_order()
    
    # Стратегии как функции
    strategies = [
        ("Стандартная", lambda o: calculate_courier_cost(o)),
        ("С дешевой доставкой", lambda o: calculate_post_cost(o)),
        ("Бесплатная", lambda o: calculate_pickup_cost(o)),
        ("Срочная", lambda o: calculate_express_cost(o)),
    ]
    
    for name, calc_func in strategies:
        processor = FunctionalOrderProcessor(calc_func)
        cost = processor.calculate_delivery_cost(order)
        print(f"\n{name}:")
        print(f"  Стоимость доставки: {cost:.2f} ₽")
    
    # Смена стратегии на лету
    processor = FunctionalOrderProcessor(calculate_courier_cost)
    print(f"\nТекущая стоимость: {processor.calculate_delivery_cost(order):.2f} ₽")
    
    # Меняем стратегию
    processor.set_calculator(calculate_pickup_cost)
    print(f"После смены: {processor.calculate_delivery_cost(order):.2f} ₽")


def demo_real_world_scenario():
    """Реальный сценарий: интернет-магазин с выбором доставки"""
    print("\n" + "=" * 70)
    print("ЧАСТЬ 5: РЕАЛЬНЫЙ СЦЕНАРИЙ — ИНТЕРНЕТ-МАГАЗИН")
    print("=" * 70)
    
    # Разные заказы для демонстрации
    orders = [
        ("Лёгкий заказ", [
            OrderItem("Книга", 0.5, Decimal('500'), 2),
            OrderItem("Ручка", 0.05, Decimal('100'), 5),
        ]),
        ("Тяжёлый заказ", [
            OrderItem("Телевизор", 15.0, Decimal('30000'), 1),
            OrderItem("Колонки", 5.0, Decimal('8000'), 2),
        ]),
        ("Дорогой заказ", [
            OrderItem("iPhone", 0.3, Decimal('80000'), 1),
            OrderItem("MacBook", 2.0, Decimal('150000'), 1),
        ]),
    ]
    
    strategies_info = [
        ("pickup", PickupDelivery()),
        ("post", PostDelivery()),
        ("courier", CourierDelivery()),
        ("express", ExpressDelivery()),
    ]
    
    for order_name, items in orders:
        address = Address("Санкт-Петербург", "Невский", "25")
        order = Order(items, address)
        
        print(f"\n{order_name}:")
        print(f"  Вес: {order.total_weight:.1f} кг")
        print(f"  Стоимость: {order.total_price:.2f} ₽")
        print("  Варианты доставки:")
        
        processor = OrderProcessor()
        for strategy_name, strategy in strategies_info:
            processor.set_strategy(strategy)
            result = processor.process_order(order)
            print(f"    • {strategy_name:8} | "
                  f"доставка: {result['delivery_cost']:7.2f} ₽ | "
                  f"время: {result['delivery_time']} | "
                  f"итого: {result['total']:8.2f} ₽")


# ============================================================
    # ГЛАВНАЯ ФУНКЦИЯ
# ============================================================

def main():
    """Главная функция демонстрации"""
    print("=" * 70)
    print("ДЕМОНСТРАЦИЯ ПАТТЕРНА STRATEGY")
    print("=" * 70)
    
    demo_classic_strategy()
    demo_strategy_switch()
    demo_factory_pattern()
    demo_functional_strategy()
    demo_real_world_scenario()
    
    print("\n" + "=" * 70)
    print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
    print("=" * 70)


if __name__ == "__main__":
    main()
Ожидаемый вывод (частичный)
text
======================================================================
ДЕМОНСТРАЦИЯ ПАТТЕРНА STRATEGY
======================================================================

======================================================================
ЧАСТЬ 1: КЛАССИЧЕСКАЯ РЕАЛИЗАЦИЯ STRATEGY
======================================================================

Заказ: 3 товаров
Вес: 3.5 кг
Стоимость товаров: 56000.00 ₽

--------------------------------------------------
СРАВНЕНИЕ СТРАТЕГИЙ ДОСТАВКИ
--------------------------------------------------

Курьерская доставка (база: 200₽ + 50₽/кг)
  Стоимость доставки: 375.00 ₽
  Время доставки: 1 day, 12:00:00
  Итого: 56375.00 ₽

Курьерская доставка (база: 300₽ + 60₽/кг)
  Стоимость доставки: 510.00 ₽
  Время доставки: 1 day, 12:00:00
  Итого: 56510.00 ₽

Почтовая доставка (база: 100₽ + 30₽/кг)
  Стоимость доставки: 205.00 ₽
  Время доставки: 5 days, 0:00:00
  Итого: 56205.00 ₽

Самовывоз (бесплатно)
  Стоимость доставки: 0.00 ₽
  Время доставки: 1:00:00
  Итого: 56000.00 ₽

Экспресс-доставка (база: 500₽ + 100₽/кг × 1.5)
  Стоимость доставки: 1275.00 ₽
  Время доставки: 3:00:00
  Итого: 57275.00 ₽
25 вариантов практической работы (ПЗ 2.9)
Общая структура каждого варианта:
Студент реализует паттерн Strategy для заданной предметной области с поддержкой минимум 3-4 стратегий.

Уровни сложности:

Варианты 1-8: базовый (простые стратегии, 3-4 типа)

Варианты 9-17: средний (стратегии с параметрами, композиция)

Варианты 18-25: сложный (динамические стратегии, фабрики, метапрограммирование)

Варианты 1-8 (Базовый уровень)
Вариант 1. Калькулятор скидок
ТЗ: Реализовать систему расчёта скидок с поддержкой разных стратегий:

Процентная скидка (5%, 10%, 15%)

Фиксированная скидка (вычитается сумма)

Скидка постоянного покупателя (зависит от количества покупок)

Сезонная скидка

Дополнительно: Возможность комбинирования скидок

Вариант 2. Система аутентификации
ТЗ: Реализовать стратегии проверки пароля:

Простая проверка (только длина)

Сложная проверка (длина + цифры + спецсимволы)

Проверка на словарь (пароль не должен быть в списке популярных)

Двухфакторная проверка

Дополнительно: Возможность добавлять новые стратегии

Вариант 3. Форматирование текста
ТЗ: Реализовать стратегии форматирования текста:

Верхний регистр

Нижний регистр

Капитализация (каждое слово с большой буквы)

Альтернативный регистр (КаЖдЫй ВтОрОй)

Дополнительно: Форматирование с учётом языка

Вариант 4. Сортировка данных
ТЗ: Реализовать стратегии сортировки:

Пузырьковая сортировка

Быстрая сортировка

Сортировка вставками

Сортировка слиянием

Дополнительно: Сравнение производительности

Вариант 5. Компрессия данных
ТЗ: Реализовать стратегии сжатия:

RLE (Run-Length Encoding)

LZW (Lempel-Ziv-Welch)

Хаффман

Простое удаление пробелов

Дополнительно: Оценка степени сжатия

Вариант 6. Генерация паролей
ТЗ: Реализовать стратегии генерации паролей:

Только цифры

Только буквы (строчные)

Буквы + цифры

Буквы + цифры + спецсимволы

Читаемые пароли (слова + цифры)

Дополнительно: Настраиваемая длина

Вариант 7. Округление чисел
ТЗ: Реализовать стратегии округления:

Математическое округление

Округление вниз

Округление вверх

Округление до указанного знака

Дополнительно: Банковское округление

Вариант 8. Кэширование
ТЗ: Реализовать стратегии кэширования:

FIFO (First In First Out)

LRU (Least Recently Used)

LFU (Least Frequently Used)

TTL (Time To Live)

Дополнительно: Очистка кэша

Варианты 9-17 (Средний уровень)
Вариант 9. Расчёт налогов
ТЗ: Реализовать стратегии расчёта налогов для интернет-магазина:

НДС (20%)

Упрощённая система (6%)

Патентная система

Страны ЕС (разные ставки)

Дополнительно: Стратегии с привязкой к региону

Вариант 10. Валидация форм
ТЗ: Реализовать стратегии валидации полей формы:

Email валидация (формат + MX запись)

Телефон (разные форматы стран)

Дата (проверка корректности)

Пароль (сложность + чёрный список)

Дополнительно: Композитная валидация

Вариант 11. Логирование
ТЗ: Реализовать стратегии логирования:

Консольный вывод

Файловое логирование (с ротацией)

JSON формат (структурированные логи)

Remote logging (HTTP, Syslog)

Дополнительно: Уровни логирования

Вариант 12. Сериализация
ТЗ: Реализовать стратегии сериализации:

JSON

XML

YAML

Pickle

MessagePack

Дополнительно: Десериализация

Вариант 13. Шифрование
ТЗ: Реализовать стратегии шифрования:

Caesar cipher

XOR шифрование

AES (через библиотеку)

Base64 (кодирование)

Дополнительно: Возможность выбора ключа

Вариант 14. Поиск подстроки
ТЗ: Реализовать стратегии поиска подстроки:

Наивный алгоритм

Алгоритм Кнута-Морриса-Пратта

Алгоритм Бойера-Мура

Regex (через re)

Дополнительно: Сравнение производительности

Вариант 15. Компрессия изображений
ТЗ: Реализовать стратегии сжатия изображений:

Сжатие с потерями (JPEG)

Сжатие без потерь (PNG)

Уменьшение разрешения

Уменьшение цветовой палитры

Дополнительно: Соотношение размера/качества

Вариант 16. Рекомендательные системы
ТЗ: Реализовать стратегии рекомендаций:

Популярные товары

Похожие товары (по категории)

Персональные (по истории покупок)

Случайные (для ознакомления)

Дополнительно: Комбинирование стратегий

Вариант 17. Экспорт данных
ТЗ: Реализовать стратегии экспорта:

CSV

Excel (через openpyxl)

PDF (через reportlab)

HTML (таблица)

Дополнительно: Настройка форматирования

Варианты 18-25 (Сложный уровень)
Вариант 18. Планировщик задач
ТЗ: Реализовать стратегии планирования задач:

FIFO (первый пришёл — первый ушёл)

Приоритетное (по приоритету задачи)

Round Robin (круговая)

Deadline (по крайнему сроку)

Дополнительно: Динамическое изменение стратегии

Вариант 19. Балансировка нагрузки
ТЗ: Реализовать стратегии балансировки:

Round Robin

Взвешенный Round Robin

Наименьшее количество соединений

IP hash

Дополнительно: Метрики эффективности

Вариант 20. Рендеринг графиков
ТЗ: Реализовать стратегии визуализации данных:

Линейный график

Столбчатая диаграмма

Круговая диаграмма

Точечная диаграмма

Дополнительно: Интерактивность

Вариант 21. Обработка платежей
ТЗ: Реализовать стратегии расчёта комиссий:

Фиксированная комиссия

Процентная комиссия

Градиентная (чем больше сумма, тем меньше процент)

С комбинацией (фикс + процент)

Дополнительно: Конвертация валют

Вариант 22. Система кэширования (расширенная)
ТЗ: Реализовать стратегии кэширования с:

Размером кэша (ограничение по памяти)

Временем жизни (TTL)

Приоритетом объектов

Ленивой загрузкой

Дополнительно: Паттерн Декоратор для стратегий

Вариант 23. Анализ тональности текста
ТЗ: Реализовать стратегии анализа тональности:

На основе словаря эмоций

На основе правил (наличие отрицаний, усилителей)

На основе машинного обучения (через библиотеку)

Гибридный подход

Дополнительно: Поддержка русского языка

Вариант 24. Система мониторинга
ТЗ: Реализовать стратегии проверки здоровья сервисов:

HTTP проверка (статус 200)

TCP проверка (порт открыт)

Проверка базы данных (запрос)

Проверка метрик (значение в диапазоне)

Дополнительно: Retry-стратегии

Вариант 25. Конвертер валют (расширенный)
ТЗ: Реализовать стратегии получения курсов валют:

Из локального кэша

Из API (внешний сервис)

Фиксированный курс (ручная настройка)

Прогнозируемый (на основе истории)

Дополнительно: Стратегии обновления курсов

Карточка студента (шаблон)
text
ПЗ 2.9. РЕАЛИЗАЦИЯ ПАТТЕРНА STRATEGY

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Предметная область: _________________________________

=== ТРЕБОВАНИЯ ===

Количество стратегий: ___
Базовый класс стратегии: _________________________________

Стратегии:
1. _________________________________
2. _________________________________
3. _________________________________
4. _________________________________

=== ДОПОЛНИТЕЛЬНЫЕ ТРЕБОВАНИЯ ===

□ Смена стратегии во время выполнения
□ Параметризованные стратегии
□ Композиция стратегий
□ Фабрика стратегий
□ Динамическая регистрация новых стратегий

=== ОТЧЁТ ===

Код реализации: _________________________________
Демонстрация работы (скриншоты): _________________________________
Сравнение стратегий (таблица): _________________________________

Дата выполнения: _____________Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Паттерн не реализован или код не работает
3 (удовлетворительно)	Реализованы 2-3 стратегии, базовая смена работает
4 (хорошо)	Реализованы все стратегии, есть смена во время выполнения
5 (отлично)	Полная реализация с фабрикой, документацией, сравнением стратегий
Шпаргалка по паттерну Strategy
python
# === КЛАССИЧЕСКАЯ РЕАЛИЗАЦИЯ ===

from abc import ABC, abstractmethod

class Strategy(ABC):
    @abstractmethod
    def execute(self, data):
        pass

class ConcreteStrategyA(Strategy):
    def execute(self, data):
        return f"Strategy A: {data}"

class ConcreteStrategyB(Strategy):
    def execute(self, data):
        return f"Strategy B: {data}"

class Context:
    def __init__(self, strategy: Strategy):
        self._strategy = strategy
    
    def set_strategy(self, strategy: Strategy):
        self._strategy = strategy
    
    def do_something(self, data):
        return self._strategy.execute(data)


# === ФУНКЦИОНАЛЬНАЯ РЕАЛИЗАЦИЯ ===

def strategy_a(data):
    return f"Strategy A: {data}"

def strategy_b(data):
    return f"Strategy B: {data}"

class FunctionalContext:
    def __init__(self, strategy):
        self._strategy = strategy
    
    def set_strategy(self, strategy):
        self._strategy = strategy
    
    def do_something(self, data):
        return self._strategy(data)


# === КОГДА ИСПОЛЬЗОВАТЬ ===
# - Много способов выполнить одно действие
# - Алгоритмы нужно менять во время выполнения
# - Нужно избежать условных операторов
# - Алгоритмы не зависят от состояния контекста


