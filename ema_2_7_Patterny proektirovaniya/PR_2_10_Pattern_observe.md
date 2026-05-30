# ПЗ 2.10. Реализация паттерна Observer

**Тема:** Поведенческие паттерны (Observer)

**Цель работы:**  
Научиться реализовывать паттерн Observer на Python, понимать, в каких ситуациях его применять, уметь проектировать системы с механизмом подписки и уведомлений.

**Время выполнения:** 90-120 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `pylint`, `radon`

```bash
pip install pylint radon
Нулевой вариант (эталонный) — Демонстрация преподавателя
Назначение: преподаватель выполняет этот вариант на занятии, показывая реализацию паттерна Observer.

Техническое задание (нулевой вариант)
Разработайте систему мониторинга фондового рынка с использованием паттерна Observer. Система должна отслеживать изменения цен акций и уведомлять подписчиков (инвесторов, трейдеров, аналитиков) о важных событиях. Подписчики могут подписываться на конкретные акции и типы событий.

Эталонная реализация
python
#!/usr/bin/env python3
"""
Модуль для мониторинга фондового рынка с использованием паттерна Observer.
"""

from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from typing import Dict, List, Set, Optional, Any, Callable
from datetime import datetime
from enum import Enum
import time
import threading
from collections import defaultdict


# ============================================================
# МОДЕЛИ ДАННЫХ
# ============================================================

class EventType(Enum):
    """Типы событий на рынке"""
    PRICE_CHANGE = "price_change"
    HIGH_VOLUME = "high_volume"
    NEWS_RELEASED = "news_released"
    DIVIDEND_ANNOUNCED = "dividend_announced"
    EARNINGS_REPORT = "earnings_report"
    ALL_TIME_HIGH = "all_time_high"
    ALL_TIME_LOW = "all_time_low"


@dataclass
class Stock:
    """Акция"""
    symbol: str
    name: str
    current_price: float
    previous_price: float = 0.0
    volume: int = 0
    change_percent: float = 0.0
    
    def update_price(self, new_price: float) -> bool:
        """Обновляет цену и возвращает флаг, была ли цена изменена"""
        if self.current_price == new_price:
            return False
        
        self.previous_price = self.current_price
        self.current_price = new_price
        self.change_percent = ((new_price - self.previous_price) / self.previous_price) * 100
        return True


@dataclass
class MarketEvent:
    """Событие на рынке"""
    event_type: EventType
    stock: Stock
    value: Any
    timestamp: datetime = field(default_factory=datetime.now)
    
    def __str__(self) -> str:
        return f"[{self.timestamp.strftime('%H:%M:%S')}] {self.event_type.value}: {self.stock.symbol} - {self.value}"


# ============================================================
    # ИНТЕРФЕЙСЫ НАБЛЮДАТЕЛЕЙ И СУБЪЕКТА
# ============================================================

class Observer(ABC):
    """Интерфейс наблюдателя"""
    
    @abstractmethod
    def update(self, event: MarketEvent) -> None:
        """Вызывается при наступлении события"""
        pass
    
    @abstractmethod
    def get_name(self) -> str:
        """Возвращает имя наблюдателя"""
        pass


class Subject(ABC):
    """Интерфейс наблюдаемого объекта"""
    
    @abstractmethod
    def attach(self, observer: Observer, stock_symbols: Optional[List[str]] = None) -> None:
        """Подписка наблюдателя"""
        pass
    
    @abstractmethod
    def detach(self, observer: Observer) -> None:
        """Отписка наблюдателя"""
        pass
    
    @abstractmethod
    def notify(self, event: MarketEvent) -> None:
        """Уведомление всех подписанных наблюдателей"""
        pass


# ============================================================
    # РЕАЛИЗАЦИЯ СУБЪЕКТА (ФОНДОВЫЙ РЫНОК)
# ============================================================

class StockMarket(Subject):
    """Фондовый рынок — наблюдаемый объект"""
    
    def __init__(self):
        self._stocks: Dict[str, Stock] = {}
        self._observers: Dict[str, Set[Observer]] = defaultdict(set)  # по типу события
        self._stock_observers: Dict[str, Set[Observer]] = defaultdict(set)  # по акции
        self._all_observers: Set[Observer] = set()
        self._history: List[MarketEvent] = []
        self._lock = threading.Lock()
    
    def add_stock(self, stock: Stock) -> None:
        """Добавление акции на рынок"""
        self._stocks[stock.symbol] = stock
        print(f"[Рынок] Добавлена акция: {stock.symbol} ({stock.name})")
    
    def update_price(self, symbol: str, new_price: float) -> None:
        """Обновление цены акции"""
        stock = self._stocks.get(symbol)
        if not stock:
            return
        
        if stock.update_price(new_price):
            # Событие: изменение цены
            self._notify_price_change(stock)
            
            # Проверка на рекордные значения
            if stock.current_price > max(self._get_price_history(symbol)):
                self._notify_all_time_high(stock)
            elif stock.current_price < min(self._get_price_history(symbol)):
                self._notify_all_time_low(stock)
    
    def update_volume(self, symbol: str, volume: int) -> None:
        """Обновление объёма торгов"""
        stock = self._stocks.get(symbol)
        if not stock:
            return
        
        stock.volume = volume
        
        # Событие: высокий объём (более 1 млн)
        if volume > 1_000_000:
            event = MarketEvent(EventType.HIGH_VOLUME, stock, volume)
            self.notify(event)
    
    def announce_dividend(self, symbol: str, amount: float) -> None:
        """Объявление дивидендов"""
        stock = self._stocks.get(symbol)
        if not stock:
            return
        
        event = MarketEvent(EventType.DIVIDEND_ANNOUNCED, stock, f"{amount}$ на акцию")
        self.notify(event)
    
    def publish_news(self, symbol: str, headline: str) -> None:
        """Публикация новости"""
        stock = self._stocks.get(symbol)
        if not stock:
            return
        
        event = MarketEvent(EventType.NEWS_RELEASED, stock, headline)
        self.notify(event)
    
    def publish_earnings(self, symbol: str, eps: float, revenue: float) -> None:
        """Публикация отчёта о доходах"""
        stock = self._stocks.get(symbol)
        if not stock:
            return
        
        event = MarketEvent(EventType.EARNINGS_REPORT, stock, f"EPS: {eps}$, Revenue: {revenue}M$")
        self.notify(event)
    
    def _notify_price_change(self, stock: Stock) -> None:
        """Уведомление об изменении цены"""
        event = MarketEvent(EventType.PRICE_CHANGE, stock, f"{stock.previous_price:.2f} → {stock.current_price:.2f} ({stock.change_percent:+.2f}%)")
        self.notify(event)
    
    def _notify_all_time_high(self, stock: Stock) -> None:
        """Уведомление об историческом максимуме"""
        event = MarketEvent(EventType.ALL_TIME_HIGH, stock, f"Новый максимум: {stock.current_price:.2f}$")
        self.notify(event)
    
    def _notify_all_time_low(self, stock: Stock) -> None:
        """Уведомление об историческом минимуме"""
        event = MarketEvent(EventType.ALL_TIME_LOW, stock, f"Новый минимум: {stock.current_price:.2f}$")
        self.notify(event)
    
    def _get_price_history(self, symbol: str) -> List[float]:
        """Получение истории цен (упрощённо)"""
        stock = self._stocks.get(symbol)
        if stock:
            return [stock.previous_price, stock.current_price]
        return [0]
    
    # ========== РЕАЛИЗАЦИЯ SUBJECT ==========
    
    def attach(self, observer: Observer, stock_symbols: Optional[List[str]] = None) -> None:
        """Подписка наблюдателя"""
        with self._lock:
            self._all_observers.add(observer)
            
            if stock_symbols:
                for symbol in stock_symbols:
                    self._stock_observers[symbol].add(observer)
                print(f"[Рынок] Подписан {observer.get_name()} на акции: {stock_symbols}")
            else:
                print(f"[Рынок] Подписан {observer.get_name()} на все события")
    
    def detach(self, observer: Observer) -> None:
        """Отписка наблюдателя"""
        with self._lock:
            self._all_observers.discard(observer)
            for symbol_set in self._stock_observers.values():
                symbol_set.discard(observer)
            print(f"[Рынок] Отписан {observer.get_name()}")
    
    def notify(self, event: MarketEvent) -> None:
        """Уведомление наблюдателей"""
        self._history.append(event)
        
        with self._lock:
            # Уведомляем всех, кто подписан на все события
            for observer in self._all_observers:
                observer.update(event)
            
            # Уведомляем подписанных на конкретную акцию
            symbol_observers = self._stock_observers.get(event.stock.symbol, set())
            for observer in symbol_observers:
                if observer not in self._all_observers:
                    observer.update(event)
    
    def get_history(self, limit: int = 10) -> List[MarketEvent]:
        """Получение истории событий"""
        return self._history[-limit:]


# ============================================================
    # РЕАЛИЗАЦИИ НАБЛЮДАТЕЛЕЙ
# ============================================================

class Investor(Observer):
    """Инвестор — долгосрочный наблюдатель"""
    
    def __init__(self, name: str, portfolio: Dict[str, int]):
        self.name = name
        self.portfolio = portfolio
        self.notifications: List[MarketEvent] = []
    
    def get_name(self) -> str:
        return f"Инвестор {self.name}"
    
    def update(self, event: MarketEvent) -> None:
        self.notifications.append(event)
        
        # Интересуется только акциями из своего портфеля
        if event.stock.symbol in self.portfolio:
            if event.event_type == EventType.DIVIDEND_ANNOUNCED:
                shares = self.portfolio[event.stock.symbol]
                print(f"  💰 {self.get_name()}: Дивиденды по {event.stock.symbol}! "
                      f"Ваши {shares} акций принесут доход!")
            elif event.event_type == EventType.EARNINGS_REPORT:
                print(f"  📊 {self.get_name()}: Отчёт {event.stock.symbol} — {event.value}")
            elif abs(event.stock.change_percent) > 10:
                print(f"  ⚠️ {self.get_name()}: {event.stock.symbol} изменилась на {event.stock.change_percent:+.1f}%!")


class Trader(Observer):
    """Трейдер — активный наблюдатель"""
    
    def __init__(self, name: str, risk_level: str = "medium"):
        self.name = name
        self.risk_level = risk_level
        self.alerts: List[MarketEvent] = []
    
    def get_name(self) -> str:
        return f"Трейдер {self.name}"
    
    def update(self, event: MarketEvent) -> None:
        self.alerts.append(event)
        
        # Трейдеров интересуют резкие движения
        if event.event_type == EventType.PRICE_CHANGE:
            change = event.stock.change_percent
            if abs(change) > 5:
                direction = "🚀 РАСТЁТ" if change > 0 else "📉 ПАДАЕТ"
                print(f"  {direction} {self.get_name()}: {event.stock.symbol} {change:+.2f}%! "
                      f"Срочно анализируем!")
        
        elif event.event_type == EventType.HIGH_VOLUME:
            print(f"  📈 {self.get_name()}: Аномальный объём по {event.stock.symbol}! "
                  f"{event.value} акций за день")
        
        elif event.event_type == EventType.NEWS_RELEASED:
            if "див" in str(event.value).lower() or "акци" in str(event.value).lower():
                print(f"  📰 {self.get_name()}: Важная новость по {event.stock.symbol}!")


class Analyst(Observer):
    """Аналитик — собирает статистику"""
    
    def __init__(self, name: str):
        self.name = name
        self.stats: Dict[str, Dict] = defaultdict(lambda: {"price_changes": [], "news_count": 0})
    
    def get_name(self) -> str:
        return f"Аналитик {self.name}"
    
    def update(self, event: MarketEvent) -> None:
        symbol = event.stock.symbol
        
        if event.event_type == EventType.PRICE_CHANGE:
            self.stats[symbol]["price_changes"].append(event.stock.change_percent)
            self.stats[symbol]["last_price"] = event.stock.current_price
        
        elif event.event_type == EventType.NEWS_RELEASED:
            self.stats[symbol]["news_count"] += 1
            self.stats[symbol]["latest_news"] = event.value
    
    def get_report(self) -> str:
        """Генерация отчёта"""
        report = f"\n📊 ОТЧЁТ АНАЛИТИКА {self.name}:\n"
        report += "-" * 40 + "\n"
        
        for symbol, data in self.stats.items():
            changes = data.get("price_changes", [])
            if changes:
                avg_change = sum(changes) / len(changes)
                report += f"{symbol}: среднее изменение {avg_change:+.2f}%, "
                report += f"новостей: {data.get('news_count', 0)}\n"
            else:
                report += f"{symbol}: данных пока нет\n"
        
        return report


class AlertSystem(Observer):
    """Система оповещения (например, для ботов)"""
    
    def __init__(self, threshold: float = 5.0):
        self.threshold = threshold
        self.alerts: List[MarketEvent] = []
    
    def get_name(self) -> str:
        return "Система оповещений"
    
    def update(self, event: MarketEvent) -> None:
        if event.event_type == EventType.PRICE_CHANGE:
            if abs(event.stock.change_percent) >= self.threshold:
                self.alerts.append(event)
                self._send_alert(event)
    
    def _send_alert(self, event: MarketEvent) -> None:
        """Отправка алерта (здесь просто вывод)"""
        icon = "🔴" if event.stock.change_percent < 0 else "🟢"
        print(f"  {icon} ALERT! {event.stock.symbol}: {event.stock.change_percent:+.2f}%")


class EmailNotifier(Observer):
    """Email-уведомления (симуляция)"""
    
    def __init__(self, email: str, symbols: Optional[List[str]] = None):
        self.email = email
        self.symbols = symbols or []
    
    def get_name(self) -> str:
        return f"EmailNotifier ({self.email})"
    
    def update(self, event: MarketEvent) -> None:
        if self.symbols and event.stock.symbol not in self.symbols:
            return
        
        if event.event_type == EventType.PRICE_CHANGE:
            print(f"  📧 [EMAIL to {self.email}] {event.stock.symbol}: {event.value}")
        elif event.event_type == EventType.DIVIDEND_ANNOUNCED:
            print(f"  📧 [EMAIL to {self.email}] {event.stock.symbol}: ДИВИДЕНДЫ! {event.value}")


class LoggerObserver(Observer):
    """Логирование всех событий"""
    
    def __init__(self, log_file: str = "market.log"):
        self.log_file = log_file
    
    def get_name(self) -> str:
        return "Logger"
    
    def update(self, event: MarketEvent) -> None:
        with open(self.log_file, "a", encoding='utf-8') as f:
            f.write(f"{event}\n")
        # Не выводим в консоль, чтобы не засорять


# ============================================================
    # ДОПОЛНИТЕЛЬНО: ОБОБЩЁННАЯ РЕАЛИЗАЦИЯ EVENT BUS
# ============================================================

class EventBus:
    """Универсальная шина событий"""
    
    def __init__(self):
        self._handlers: Dict[EventType, List[Callable]] = defaultdict(list)
    
    def subscribe(self, event_type: EventType, handler: Callable) -> None:
        """Подписка на событие"""
        self._handlers[event_type].append(handler)
    
    def unsubscribe(self, event_type: EventType, handler: Callable) -> None:
        """Отписка от события"""
        if handler in self._handlers[event_type]:
            self._handlers[event_type].remove(handler)
    
    def emit(self, event_type: EventType, data: Any) -> None:
        """Генерация события"""
        for handler in self._handlers[event_type]:
            handler(data)
    
    def emit_async(self, event_type: EventType, data: Any) -> None:
        """Асинхронная генерация события"""
        thread = threading.Thread(target=self.emit, args=(event_type, data))
        thread.start()


# ============================================================
    # ДЕМОНСТРАЦИЯ
# ============================================================

def demo_classic_observer():
    """Демонстрация классической реализации Observer"""
    print("\n" + "=" * 70)
    print("ЧАСТЬ 1: КЛАССИЧЕСКАЯ РЕАЛИЗАЦИЯ OBSERVER")
    print("=" * 70)
    
    # Создаём рынок
    market = StockMarket()
    
    # Добавляем акции
    market.add_stock(Stock("AAPL", "Apple Inc.", 150.0, 145.0))
    market.add_stock(Stock("GOOGL", "Alphabet Inc.", 140.0, 138.0))
    market.add_stock(Stock("TSLA", "Tesla Inc.", 250.0, 240.0))
    
    # Создаём наблюдателей
    warren = Investor("Уоррен Баффет", {"AAPL": 1000, "GOOGL": 500})
    elon_trader = Trader("Элон Трейдер", risk_level="high")
    jane_analyst = Analyst("Джейн")
    alert_system = AlertSystem(threshold=3.0)
    email_notifier = EmailNotifier("investor@example.com", ["AAPL", "TSLA"])
    logger = LoggerObserver()
    
    # Подписываем наблюдателей
    print("\n--- ПОДПИСКА ---")
    market.attach(warren)                     # на все события
    market.attach(elon_trader, ["AAPL", "TSLA"])  # только на определённые акции
    market.attach(jane_analyst)               # на все (сбор статистики)
    market.attach(alert_system)               # на все
    market.attach(email_notifier)             # только на AAPL и TSLA
    market.attach(logger)                     # логирование (без вывода)
    
    # Симуляция событий
    print("\n--- СИМУЛЯЦИЯ РЫНОЧНЫХ СОБЫТИЙ ---")
    
    # Обновление цен
    print("\n1. Изменение цен:")
    market.update_price("AAPL", 155.0)
    market.update_price("AAPL", 162.0)
    market.update_price("TSLA", 260.0)
    market.update_price("GOOGL", 135.0)
    
    # Аномальный объём
    print("\n2. Аномальный объём торгов:")
    market.update_volume("TSLA", 1_500_000)
    
    # Новости
    print("\n3. Публикация новостей:")
    market.publish_news("AAPL", "Apple announces new AI features")
    market.publish_news("TSLA", "Tesla expands Gigafactory in Germany")
    
    # Дивиденды
    print("\n4. Объявление дивидендов:")
    market.announce_dividend("AAPL", 0.95)
    
    # Отчёты о доходах
    print("\n5. Публикация отчётов:")
    market.publish_earnings("GOOGL", 1.45, 80.5)
    
    # Отписка
    print("\n--- ОТПИСКА ---")
    market.detach(email_notifier)
    
    print("\n6. После отписки (изменения не должны приходить на email):")
    market.update_price("AAPL", 170.0)
    market.announce_dividend("AAPL", 1.05)
    
    # Отчёт аналитика
    print(jane_analyst.get_report())


def demo_filtered_observers():
    """Демонстрация фильтрации событий"""
    print("\n" + "=" * 70)
    print("ЧАСТЬ 2: ФИЛЬТРАЦИЯ СОБЫТИЙ")
    print("=" * 70)
    
    market = StockMarket()
    market.add_stock(Stock("AAPL", "Apple Inc.", 150.0))
    market.add_stock(Stock("MSFT", "Microsoft", 330.0))
    market.add_stock(Stock("AMZN", "Amazon", 130.0))
    
    class StockSpecificObserver(Observer):
        def __init__(self, name: str, interested_symbols: List[str]):
            self.name = name
            self.interested_symbols = interested_symbols
        
        def get_name(self) -> str:
            return self.name
        
        def update(self, event: MarketEvent) -> None:
            if event.stock.symbol in self.interested_symbols:
                print(f"  {self.name}: {event.stock.symbol} — {event.value}")
    
    # Создаём наблюдателей с разными интересами
    apple_only = StockSpecificObserver("Только Apple", ["AAPL"])
    tech_stocks = StockSpecificObserver("Tech-акции", ["AAPL", "MSFT"])
    all_stocks = StockSpecificObserver("Все акции", ["AAPL", "MSFT", "AMZN"])
    
    market.attach(apple_only)
    market.attach(tech_stocks)
    market.attach(all_stocks)
    
    print("\nСимуляция изменений:")
    market.update_price("AAPL", 155.0)
    market.update_price("MSFT", 335.0)
    market.update_price("AMZN", 135.0)


def demo_event_bus():
    """Демонстрация универсальной шины событий"""
    print("\n" + "=" * 70)
    print("ЧАСТЬ 3: УНИВЕРСАЛЬНАЯ ШИНА СОБЫТИЙ (EVENT BUS)")
    print("=" * 70)
    
    bus = EventBus()
    
    # Простые обработчики
    def on_price_change(data):
        print(f"  Цена изменена: {data}")
    
    def on_news(data):
        print(f"  НОВОСТЬ: {data}")
    
    def on_error(data):
        print(f"  ОШИБКА: {data}")
    
    # Подписка
    bus.subscribe(EventType.PRICE_CHANGE, on_price_change)
    bus.subscribe(EventType.NEWS_RELEASED, on_news)
    bus.subscribe(EventType.PRICE_CHANGE, on_error)  # ошибочная подписка
    
    # Генерация событий
    print("\nГенерация событий:")
    bus.emit(EventType.PRICE_CHANGE, "AAPL: 150 → 155")
    bus.emit(EventType.NEWS_RELEASED, "Apple представила новый iPhone")
    
    # Отписка
    bus.unsubscribe(EventType.PRICE_CHANGE, on_error)
    print("\nПосле отписки обработчика ошибок:")
    bus.emit(EventType.PRICE_CHANGE, "GOOGL: 140 → 142")


def demo_async_observer():
    """Демонстрация асинхронных уведомлений"""
    print("\n" + "=" * 70)
    print("ЧАСТЬ 4: АСИНХРОННЫЕ УВЕДОМЛЕНИЯ")
    print("=" * 70)
    
    class AsyncObserver(Observer):
        def __init__(self, name: str, delay: float = 0.5):
            self.name = name
            self.delay = delay
        
        def get_name(self) -> str:
            return self.name
        
        def update(self, event: MarketEvent) -> None:
            # Симуляция долгой обработки
            time.sleep(self.delay)
            print(f"  {self.name} обработал {event.event_type.value} для {event.stock.symbol}")
    
    market = StockMarket()
    market.add_stock(Stock("TEST", "Test Stock", 100.0))
    
    # Создаём "медленных" наблюдателей
    slow_observer = AsyncObserver("Медленный обработчик", delay=0.3)
    market.attach(slow_observer)
    
    print("\nСобытие (обработка занимает время):")
    start = time.time()
    market.update_price("TEST", 105.0)
    elapsed = time.time() - start
    print(f"Все уведомления обработаны за {elapsed:.2f} сек")


def demo_real_world_scenario():
    """Реальный сценарий: торговый бот с несколькими стратегиями"""
    print("\n" + "=" * 70)
    print("ЧАСТЬ 5: РЕАЛЬНЫЙ СЦЕНАРИЙ — ТОРГОВЫЙ БОТ")
    print("=" * 70)
    
    class TradingBot(Observer):
        def __init__(self, name: str, strategy: str):
            self.name = name
            self.strategy = strategy
            self.signals: List[dict] = []
        
        def get_name(self) -> str:
            return f"Бот {self.name} ({self.strategy})"
        
        def update(self, event: MarketEvent) -> None:
            if event.event_type != EventType.PRICE_CHANGE:
                return
            
            signal = None
            if self.strategy == "momentum" and abs(event.stock.change_percent) > 5:
                signal = {"action": "BUY" if event.stock.change_percent > 0 else "SELL", "symbol": event.stock.symbol}
            elif self.strategy == "reversal" and abs(event.stock.change_percent) > 3:
                signal = {"action": "BUY" if event.stock.change_percent < 0 else "SELL", "symbol": event.stock.symbol}
            elif self.strategy == "scalper":
                signal = {"action": "BUY", "symbol": event.stock.symbol}
            
            if signal:
                self.signals.append(signal)
                print(f"  🤖 {self.get_name()}: {signal['action']} {signal['symbol']}")
    
    market = StockMarket()
    market.add_stock(Stock("AAPL", "Apple", 150.0))
    market.add_stock(Stock("MSFT", "Microsoft", 330.0))
    market.add_stock(Stock("TSLA", "Tesla", 250.0))
    
    # Создаём ботов с разными стратегиями
    momentum_bot = TradingBot("MomentumBot", "momentum")
    reversal_bot = TradingBot("ReversalBot", "reversal")
    scalper_bot = TradingBot("ScalperBot", "scalper")
    
    market.attach(momentum_bot)
    market.attach(reversal_bot)
    market.attach(scalper_bot)
    
    print("\nСимуляция торговой сессии:")
    
    # Симуляция изменений цен
    changes = [
        ("AAPL", 158.0),   # +5.33%
        ("MSFT", 335.0),   # +1.5%
        ("TSLA", 240.0),   # -4%
        ("AAPL", 165.0),   # +4.4%
        ("TSLA", 230.0),   # -4.2%
        ("MSFT", 320.0),   # -3%
    ]
    
    for symbol, price in changes:
        print(f"\n{symbol}: {price}")
        market.update_price(symbol, price)
    
    print(f"\nСтатистика ботов:")
    print(f"  MomentumBot сигналов: {len(momentum_bot.signals)}")
    print(f"  ReversalBot сигналов: {len(reversal_bot.signals)}")
    print(f"  ScalperBot сигналов: {len(scalper_bot.signals)}")


# ============================================================
    # ГЛАВНАЯ ФУНКЦИЯ
# ============================================================

def main():
    """Главная функция демонстрации"""
    print("=" * 70)
    print("ДЕМОНСТРАЦИЯ ПАТТЕРНА OBSERVER")
    print("=" * 70)
    
    demo_classic_observer()
    demo_filtered_observers()
    demo_event_bus()
    demo_async_observer()
    demo_real_world_scenario()
    
    print("\n" + "=" * 70)
    print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
    print("=" * 70)


if __name__ == "__main__":
    main()
Ожидаемый вывод (частичный)
text
======================================================================
ДЕМОНСТРАЦИЯ ПАТТЕРНА OBSERVER
======================================================================

======================================================================
ЧАСТЬ 1: КЛАССИЧЕСКАЯ РЕАЛИЗАЦИЯ OBSERVER
======================================================================

[Рынок] Добавлена акция: AAPL (Apple Inc.)
[Рынок] Добавлена акция: GOOGL (Alphabet Inc.)
[Рынок] Добавлена акция: TSLA (Tesla Inc.)

--- ПОДПИСКА ---
[Рынок] Подписан Инвестор Уоррен Баффет на все события
[Рынок] Подписан Трейдер Элон Трейдер на акции: ['AAPL', 'TSLA']
[Рынок] Подписан Аналитик Джейн на все события
[Рынок] Подписан Система оповещений на все события
[Рынок] Подписан EmailNotifier (investor@example.com) на все события
[Рынок] Подписан Logger на все события

--- СИМУЛЯЦИЯ РЫНОЧНЫХ СОБЫТИЙ ---

1. Изменение цен:
  ⚠️ Инвестор Уоррен Баффет: AAPL изменилась на +3.4%!
  📈 Трейдер Элон Трейдер: AAPL +3.45%! Срочно анализируем!
  🟢 ALERT! AAPL: +3.45%
  📧 [EMAIL to investor@example.com] AAPL: 145.0 → 150.0 (+3.45%)

...

2. Аномальный объём торгов:
  📈 Трейдер Элон Трейдер: Аномальный объём по TSLA! 1500000 акций за день
  📧 [EMAIL to investor@example.com] TSLA: ДИВИДЕНДЫ! 0.95$ на акцию

...

======================================================================
ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА
======================================================================
25 вариантов практической работы (ПЗ 2.10)
Общая структура каждого варианта:
Студент реализует паттерн Observer для заданной предметной области с поддержкой минимум 3-4 наблюдателей и различных типов событий.

Уровни сложности:

Варианты 1-8: базовый (простые наблюдатели, 3-4 типа событий)

Варианты 9-17: средний (фильтрация событий, приоритеты)

Варианты 18-25: сложный (асинхронность, распределённые системы)

Варианты 1-8 (Базовый уровень)
Вариант 1. Система уведомлений о погоде
ТЗ: Реализовать метеостанцию с наблюдателями:

Метеостанция (Subject): температура, давление, влажность

Наблюдатели: Дисплей текущих условий, Статистика, Прогноз погоды

События: изменение температуры, изменение давления, изменение влажности

Вариант 2. Система оповещения о курсе валют
ТЗ: Реализовать систему слежения за курсом валют:

Курс валют (Subject): USD/RUB, EUR/RUB

Наблюдатели: Трейдер (алерты при резких изменениях), Банк (логирование), Обычный пользователь (ежедневный отчёт)

События: изменение курса, достижение порога

Вариант 3. Система контроля температуры
ТЗ: Реализовать систему мониторинга температуры:

Датчик температуры (Subject)

Наблюдатели: Сигнализация (при превышении), Логгер, Дисплей, Система отопления

События: изменение температуры, превышение порога

Вариант 4. Новостной агрегатор
ТЗ: Реализовать новостную ленту:

Новостной портал (Subject): новые статьи

Наблюдатели: Email-подписчик, RSS-читатель, Мобильное приложение, Архиватор

События: новая новость, обновление категории
Вариант 5. Система мониторинга серверов
ТЗ: Реализовать систему мониторинга серверов:

Сервер (Subject): CPU, память, диск

Наблюдатели: Администратор (алерты), Логгер, График метрик, Автоматическое масштабирование

События: высокая нагрузка, перегрузка, восстановление

Вариант 6. Система контроля запасов
ТЗ: Реализовать систему учёта товаров:

Склад (Subject): остатки товаров

Наблюдатели: Менеджер (при низком остатке), Система заказов, Отдел закупок, Бухгалтерия

События: низкий остаток, пополнение, списание

Вариант 7. Система уведомлений о задачах
ТЗ: Реализовать систему управления задачами:

Трекер задач (Subject): статусы задач

Наблюдатели: Исполнитель, Руководитель, Система напоминаний, Аналитик

События: создание задачи, изменение статуса, дедлайн

Вариант 8. Социальная сеть (уведомления)
ТЗ: Реализовать систему уведомлений:

Пользователь (Subject): посты, лайки, комментарии

Наблюдатели: Друзья (лента), Система уведомлений, Аналитика, Логгер

События: новый пост, новый лайк, новый комментарий

Варианты 9-17 (Средний уровень)
Вариант 9. Торговая платформа
ТЗ: Реализовать систему торговли с несколькими стратегиями:

Биржа (Subject): цены акций, объёмы

Наблюдатели: Маркет-мейкер, Арбитражный бот, Трейдер-алгоритм, Система рисков

Дополнительно: Подписка на конкретные акции, фильтрация по объёму

Вариант 10. Система умного дома
ТЗ: Реализовать систему умного дома:

Центральный контроллер (Subject): датчики (температура, свет, движение)

Наблюдатели: Кондиционер, Освещение, Сигнализация, Мобильное приложение

Дополнительно: Приоритеты уведомлений, сценарии

Вариант 11. Система логирования с ротацией
ТЗ: Реализовать распределённую систему логирования:

Логгер (Subject): события приложения

Наблюдатели: Консоль, Файл с ротацией, БД, Система мониторинга

Дополнительно: Фильтрация по уровню (INFO, WARNING, ERROR)

Вариант 12. Система очередей
ТЗ: Реализовать систему очередей сообщений:

Очередь (Subject): входящие сообщения

Наблюдатели: Воркер 1, Воркер 2, Dead Letter Queue, Монитор

Дополнительно: Балансировка нагрузки между воркерами

Вариант 13. Система ставок на спорт
ТЗ: Реализовать систему ставок:

Матч (Subject): счёт, время

Наблюдатели: Букмекер, Игроки, Статистика, Система выплат

Дополнительно: Изменение коэффициентов в реальном времени

Вариант 14. Система голосования
ТЗ: Реализовать систему онлайн-голосования:

Голосование (Subject): результаты

Наблюдатели: Экран результатов, Аналитики, Участники, Архив

Дополнительно: Анонимное голосование, подсчёт

Вариант 15. Система CI/CD
ТЗ: Реализовать систему непрерывной интеграции:

CI-сервер (Subject): сборки, тесты

Наблюдатели: Разработчики, Система деплоя, Отчёт, Логгер

Дополнительно: Статусы (успех/провал), уведомления в чат

Вариант 16. Система чата
ТЗ: Реализовать систему чата с комнатами:

Чат-комната (Subject): сообщения

Наблюдатели: Участники, Модераторы, Бот-логгер, Архиватор

Дополнительно: Приватные сообщения, команды ботов

Вариант 17. Система доставки еды
ТЗ: Реализовать систему отслеживания заказа:

Заказ (Subject): статус заказа

Наблюдатели: Клиент, Ресторан, Курьер, Диспетчер

Дополнительно: Уведомления на каждом этапе

Варианты 18-25 (Сложный уровень)
Вариант 18. Распределённый Event Bus
ТЗ: Реализовать распределённую шину событий:

Поддержка нескольких узлов

Асинхронная обработка

Гарантия доставки (at-least-once)

Мониторинг очередей

Вариант 19. Система с приоритетами
ТЗ: Реализовать систему с приоритетными уведомлениями:

Наблюдатели с разными приоритетами (критичный, высокий, обычный)

Приоритетная очередь событий

Дроп низкоприоритетных при перегрузке

Вариант 20. Плагинная архитектура
ТЗ: Реализовать систему с динамической загрузкой наблюдателей:

Наблюдатели загружаются как плагины

Включение/выключение во время работы

Динамическая регистрация событий

Вариант 21. Система с подпиской на паттерны
ТЗ: Реализовать сложную систему фильтрации:

Подписка по регулярным выражениям (названия акций)

Подписка по диапазонам значений

Композитные условия (AND, OR, NOT)

Вариант 22. Отложенные уведомления
ТЗ: Реализовать систему с отложенными уведомлениями:

Уведомления можно отложить на время

Scheduled-события

Повторные попытки при ошибке

Вариант 23. Система с транзакционностью
ТЗ: Реализовать Observer с транзакционной поддержкой:

Групповые уведомления

Откат при ошибке

ACID-свойства (атомарность)

Вариант 24. Кроссплатформенный Observer
ТЗ: Реализовать Observer для распределённой системы:

WebSocket уведомления для веб-клиентов

gRPC для микросервисов

Redis Pub/Sub для масштабирования

Вариант 25. Система мониторинга блокчейна
ТЗ: Реализовать систему мониторинга транзакций:

Отслеживание новых блоков

Наблюдатели для разных кошельков

Анализ транзакций

Уведомления о подтверждениях

Карточка студента (шаблон)
text
ПЗ 2.10. РЕАЛИЗАЦИЯ ПАТТЕРНА OBSERVER

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Предметная область: _________________________________

=== ТРЕБОВАНИЯ ===

Субъект (Subject): _________________________________
Типы событий:
1. _________________________________
2. _________________________________
3. _________________________________

Наблюдатели (Observer):
1. _________________________________
2. _________________________________
3. _________________________________

=== ДОПОЛНИТЕЛЬНЫЕ ТРЕБОВАНИЯ ===

□ Фильтрация событий
□ Приоритеты наблюдателей
□ Асинхронные уведомления
□ Подписка на конкретные типы
□ Динамическая подписка/отписка

=== ОТЧЁТ ===

Диаграмма классов: _________________________________
Код реализации: _________________________________
Демонстрация работы (скриншоты): _________________________________

Дата выполнения: _____________
Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Паттерн не реализован или код не работает
3 (удовлетворительно)	Реализованы 2-3 наблюдателя, базовая подписка работает
4 (хорошо)	Реализованы все наблюдатели, есть фильтрация событий
5 (отлично)	Полная реализация с динамической подпиской, асинхронностью, документацией
Шпаргалка по паттерну Observer
python
# === КЛАССИЧЕСКАЯ РЕАЛИЗАЦИЯ ===

from abc import ABC, abstractmethod
from typing import List

class Observer(ABC):
    @abstractmethod
    def update(self, data: Any) -> None:
        pass

class Subject(ABC):
    @abstractmethod
    def attach(self, observer: Observer) -> None:
        pass
    
    @abstractmethod
    def detach(self, observer: Observer) -> None:
        pass
    
    @abstractmethod
    def notify(self) -> None:
        pass

class ConcreteSubject(Subject):
    def __init__(self):
        self._observers: List[Observer] = []
        self._state: Any = None
    
    def attach(self, observer: Observer) -> None:
        self._observers.append(observer)
    
    def detach(self, observer: Observer) -> None:
        self._observers.remove(observer)
    
    def notify(self) -> None:
        for observer in self._observers:
            observer.update(self._state)
    
    def set_state(self, state: Any) -> None:
        self._state = state
        self.notify()


# === ФУНКЦИОНАЛЬНАЯ РЕАЛИЗАЦИЯ ===

class EventBus:
    def __init__(self):
        self._handlers: Dict[str, List[Callable]] = {}
    
    def subscribe(self, event: str, handler: Callable) -> None:
        self._handlers.setdefault(event, []).append(handler)
    
    def emit(self, event: str, data: Any) -> None:
        for handler in self._handlers.get(event, []):
            handler(data)


# === КОГДА ИСПОЛЬЗОВАТЬ ===
# - Изменение одного объекта влияет на другие
# - Количество наблюдателей неизвестно
# - Нужно слабое связывание
# - События в UI, логировании, мониторинге
