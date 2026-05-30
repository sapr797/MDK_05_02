# ПЗ 2.15. Создание иерархии пользовательских исключений

**Тема:** Обработка исключений, создание собственных исключений, иерархия классов

**Цель работы:**  
Научиться создавать собственную иерархию исключений для предметной области, правильно использовать наследование, перехватывать и обрабатывать пользовательские исключения.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+

> Главная мысль: **Стандартные исключения — как универсальные инструменты. Свои исключения — как специальные инструменты для конкретной задачи. Правильная иерархия упрощает отладку и обработку ошибок.**

---

## Содержание

1. [Теоретическая справка](#1-теоретическая-справка)
2. [Нулевой вариант (эталонный)](#2-нулевой-вариант-эталонный)
3. [25 вариантов практической работы](#3-25-вариантов-практической-работы)
4. [Критерии оценки](#4-критерии-оценки)
5. [Шпаргалка](#5-шпаргалка)

---

## 1. Теоретическая справка

### 1.1. Что такое пользовательские исключения

**Пользовательские исключения** — это классы, наследующиеся от `Exception`, которые позволяют моделировать специфические ошибки предметной области.

```python
# Простейшее пользовательское исключение
class MyAppError(Exception):
    pass

# Использование
raise MyAppError("Что-то пошло не так")
1.2. Зачем нужна иерархия исключений
Без иерархии	С иерархией
Много независимых исключений	Чёткая структура родитель-потомок
Сложно перехватывать группы ошибок	Можно перехватывать по категориям
Дублирование логики обработки	Общая логика в базовом классе
Непонятна связь между ошибками	Ясная семантика ошибок
1.3. Правила создания иерархии
python
# Правильная иерархия
class AppError(Exception):
    """Базовое исключение приложения."""
    pass

class DatabaseError(AppError):
    """Ошибки базы данных."""
    pass

class ConnectionError(DatabaseError):
    """Ошибки подключения."""
    pass

class QueryError(DatabaseError):
    """Ошибки запросов."""
    pass

class ValidationError(AppError):
    """Ошибки валидации."""
    pass

# Использование
try:
    # какой-то код
    pass
except ConnectionError:
    print("Проблемы с сетью")
except DatabaseError:
    print("Ошибка БД (но не сеть)")
except AppError:
    print("Любая ошибка приложения")
1.4. Добавление атрибутов в исключения
python
class ValidationError(Exception):
    """Исключение с дополнительными атрибутами."""
    
    def __init__(self, message: str, field: str = None, value: any = None):
        self.message = message
        self.field = field
        self.value = value
        super().__init__(message)
    
    def __str__(self) -> str:
        if self.field:
            return f"[{self.field}] {self.message} (значение: {self.value})"
        return self.message


# Использование
try:
    raise ValidationError("Некорректный email", "email", "invalid")
except ValidationError as e:
    print(f"Ошибка в поле {e.field}: {e.message}")
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая создание иерархии исключений.

Техническое задание (нулевой вариант)
Разработайте иерархию исключений для банковской системы. Создайте базовое исключение BankError и производные:

AccountError (ошибки счёта): InsufficientFundsError, AccountNotFoundError, AccountBlockedError

TransactionError (ошибки транзакций): InvalidAmountError, DailyLimitError

AuthError (ошибки авторизации): InvalidPinError, CardExpiredError

Эталонная реализация
python
"""
bank_exceptions.py — Иерархия исключений для банковской системы.
"""

from datetime import date
from typing import Optional, Any


# ============================================================
# БАЗОВОЕ ИСКЛЮЧЕНИЕ
# ============================================================

class BankError(Exception):
    """
    Базовое исключение для всех ошибок банковской системы.
    
    Атрибуты:
        message: Сообщение об ошибке
        code: Код ошибки (для API)
        details: Дополнительная информация
    """
    
    def __init__(self, message: str, code: int = None, details: Any = None):
        self.message = message
        self.code = code or self._default_code()
        self.details = details
        super().__init__(message)
    
    def _default_code(self) -> int:
        """Возвращает код ошибки по умолчанию."""
        return 1000
    
    def __str__(self) -> str:
        result = f"[{self.code}] {self.message}"
        if self.details:
            result += f" (Детали: {self.details})"
        return result


# ============================================================
# ОШИБКИ СЧЁТА (AccountError)
# ============================================================

class AccountError(BankError):
    """Базовое исключение для ошибок, связанных со счётом."""
    
    def _default_code(self) -> int:
        return 2000


class InsufficientFundsError(AccountError):
    """Недостаточно средств на счету."""
    
    def __init__(self, required: float, available: float, currency: str = "RUB"):
        self.required = required
        self.available = available
        self.currency = currency
        message = f"Недостаточно средств. Требуется: {required} {currency}, доступно: {available} {currency}"
        super().__init__(message, code=2001, details={"required": required, "available": available})
    
    def _default_code(self) -> int:
        return 2001


class AccountNotFoundError(AccountError):
    """Счёт не найден."""
    
    def __init__(self, account_id: str):
        self.account_id = account_id
        message = f"Счёт {account_id} не найден"
        super().__init__(message, code=2002, details={"account_id": account_id})
    
    def _default_code(self) -> int:
        return 2002


class AccountBlockedError(AccountError):
    """Счёт заблокирован."""
    
    def __init__(self, account_id: str, reason: str = None):
        self.account_id = account_id
        self.reason = reason
        message = f"Счёт {account_id} заблокирован"
        if reason:
            message += f". Причина: {reason}"
        super().__init__(message, code=2003, details={"account_id": account_id, "reason": reason})
    
    def _default_code(self) -> int:
        return 2003


# ============================================================
# ОШИБКИ ТРАНЗАКЦИЙ (TransactionError)
# ============================================================

class TransactionError(BankError):
    """Базовое исключение для ошибок транзакций."""
    
    def _default_code(self) -> int:
        return 3000


class InvalidAmountError(TransactionError):
    """Некорректная сумма транзакции."""
    
    def __init__(self, amount: float, reason: str = "Сумма должна быть положительной"):
        self.amount = amount
        self.reason = reason
        message = f"Некорректная сумма {amount}. {reason}"
        super().__init__(message, code=3001, details={"amount": amount, "reason": reason})
    
    def _default_code(self) -> int:
        return 3001


class DailyLimitError(TransactionError):
    """Превышен дневной лимит."""
    
    def __init__(self, limit: float, attempted: float, remaining: float):
        self.limit = limit
        self.attempted = attempted
        self.remaining = remaining
        message = f"Превышен дневной лимит. Лимит: {limit}, попытка: {attempted}, осталось: {remaining}"
        super().__init__(message, code=3002, details={
            "limit": limit, "attempted": attempted, "remaining": remaining
        })
    
    def _default_code(self) -> int:
        return 3002


# ============================================================
# ОШИБКИ АВТОРИЗАЦИИ (AuthError)
# ============================================================

class AuthError(BankError):
    """Базовое исключение для ошибок авторизации."""
    
    def _default_code(self) -> int:
        return 4000


class InvalidPinError(AuthError):
    """Неверный PIN-код."""
    
    def __init__(self, attempts_left: int = None):
        self.attempts_left = attempts_left
        message = "Неверный PIN-код"
        if attempts_left is not None:
            message += f". Осталось попыток: {attempts_left}"
        super().__init__(message, code=4001, details={"attempts_left": attempts_left})
    
    def _default_code(self) -> int:
        return 4001


class CardExpiredError(AuthError):
    """Срок действия карты истёк."""
    
    def __init__(self, expiry_date: date, card_number: str = None):
        self.expiry_date = expiry_date
        self.card_number = card_number
        message = f"Срок действия карты истёк {expiry_date}"
        if card_number:
            message = f"Карта {card_number[-4:]}: {message}"
        super().__init__(message, code=4002, details={
            "expiry_date": expiry_date.isoformat(),
            "card_number": card_number
        })
    
    def _default_code(self) -> int:
        return 4002


# ============================================================
# ПРИМЕР ИСПОЛЬЗОВАНИЯ
# ============================================================

class BankAccount:
    """Модель банковского счёта с использованием исключений."""
    
    def __init__(self, account_id: str, balance: float = 0.0, is_blocked: bool = False):
        self.account_id = account_id
        self.balance = balance
        self.is_blocked = is_blocked
        self.daily_withdrawn = 0.0
        self.daily_limit = 100000.0
        self.pin = "1234"
    
    def withdraw(self, amount: float, pin: str) -> float:
        """
        Снятие средств со счёта.
        
        Args:
            amount: Сумма для снятия
            pin: PIN-код
        
        Returns:
            Остаток на счёте
        
        Raises:
            AccountBlockedError: Если счёт заблокирован
            InvalidPinError: Если PIN-код неверный
            InvalidAmountError: Если сумма некорректна
            InsufficientFundsError: Если недостаточно средств
            DailyLimitError: Если превышен дневной лимит
        """
        # Проверка блокировки
        if self.is_blocked:
            raise AccountBlockedError(self.account_id, "Счёт заблокирован службой безопасности")
        
        # Проверка PIN
        if pin != self.pin:
            raise InvalidPinError(attempts_left=2)
        
        # Проверка суммы
        if amount <= 0:
            raise InvalidAmountError(amount, "Сумма должна быть больше нуля")
        
        # Проверка дневного лимита
        if self.daily_withdrawn + amount > self.daily_limit:
            raise DailyLimitError(
                limit=self.daily_limit,
                attempted=amount,
                remaining=self.daily_limit - self.daily_withdrawn
            )
        
        # Проверка достаточности средств
        if amount > self.balance:
            raise InsufficientFundsError(amount, self.balance)
        
        # Выполнение операции
        self.balance -= amount
        self.daily_withdrawn += amount
        return self.balance
    
    def deposit(self, amount: float) -> float:
        """Пополнение счёта."""
        if amount <= 0:
            raise InvalidAmountError(amount, "Сумма должна быть больше нуля")
        
        if self.is_blocked:
            raise AccountBlockedError(self.account_id, "Нельзя пополнить заблокированный счёт")
        
        self.balance += amount
        return self.balance


def demo():
    """Демонстрация работы иерархии исключений."""
    print("=" * 60)
    print("БАНКОВСКАЯ СИСТЕМА — ДЕМОНСТРАЦИЯ ИСКЛЮЧЕНИЙ")
    print("=" * 60)
    
    # Создаём счёт
    account = BankAccount("ACC-12345", balance=50000.0)
    
    print(f"\nСчёт: {account.account_id}")
    print(f"Баланс: {account.balance} RUB")
    print(f"Дневной лимит: {account.daily_limit} RUB")
    
    # Тест 1: Успешная операция
    print("\n--- Тест 1: Успешное снятие ---")
    try:
        new_balance = account.withdraw(10000, "1234")
        print(f"Снято 10000 RUB. Новый баланс: {new_balance} RUB")
    except BankError as e:
        print(f"Ошибка: {e}")
    
    # Тест 2: Неверный PIN
    print("\n--- Тест 2: Неверный PIN ---")
    try:
        account.withdraw(5000, "0000")
    except InvalidPinError as e:
        print(f"Ожидаемая ошибка: {e}")
    except BankError as e:
        print(f"Неожиданная ошибка: {e}")
    
    # Тест 3: Недостаточно средств
    print("\n--- Тест 3: Недостаточно средств ---")
    try:
        account.withdraw(100000, "1234")
    except InsufficientFundsError as e:
        print(f"Ожидаемая ошибка: {e}")
    except BankError as e:
        print(f"Неожиданная ошибка: {e}")
    
    # Тест 4: Некорректная сумма
    print("\n--- Тест 4: Некорректная сумма ---")
    try:
        account.withdraw(-500, "1234")
    except InvalidAmountError as e:
        print(f"Ожидаемая ошибка: {e}")
    except BankError as e:
        print(f"Неожиданная ошибка: {e}")
    
    # Тест 5: Перехват всех ошибок BankError
    print("\n--- Тест 5: Обработка всех банковских ошибок ---")
    try:
        account.withdraw(150000, "1234")  # Превышение лимита
    except DailyLimitError as e:
        print(f"Дневной лимит: {e}")
    except BankError as e:
        print(f"Другая банковская ошибка: {e}")
    
    # Тест 6: Иерархический перехват
    print("\n--- Тест 6: Иерархический перехват ---")
    try:
        account.is_blocked = True
        account.withdraw(1000, "1234")
    except AccountBlockedError as e:
        print(f"Поймано AccountBlockedError: {e}")
    except AccountError as e:
        print(f"Поймано AccountError: {e}")
    except BankError as e:
        print(f"Поймано BankError: {e}")


if __name__ == "__main__":
    demo()
Ожидаемый вывод
text
============================================================
БАНКОВСКАЯ СИСТЕМА — ДЕМОНСТРАЦИЯ ИСКЛЮЧЕНИЙ
============================================================

Счёт: ACC-12345
Баланс: 50000.0 RUB
Дневной лимит: 100000.0 RUB

--- Тест 1: Успешное снятие ---
Снято 10000 RUB. Новый баланс: 40000.0 RUB

--- Тест 2: Неверный PIN ---
Ожидаемая ошибка: [4001] Неверный PIN-код. Осталось попыток: 2

--- Тест 3: Недостаточно средств ---
Ожидаемая ошибка: [2001] Недостаточно средств. Требуется: 100000.0 RUB, доступно: 40000.0 RUB

--- Тест 4: Некорректная сумма ---
Ожидаемая ошибка: [3001] Некорректная сумма -500.0. Сумма должна быть больше нуля

--- Тест 5: Обработка всех банковских ошибок ---
Дневной лимит: [3002] Превышен дневной лимит. Лимит: 100000.0, попытка: 150000.0, осталось: 90000.0

--- Тест 6: Иерархический перехват ---
Поймано AccountBlockedError: [2003] Счёт ACC-12345 заблокирован. Причина: Счёт заблокирован службой безопасности
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (3-4 исключения, простая иерархия)

Варианты 9-17: средний (5-6 исключений, атрибуты, наследование)

Варианты 18-25: сложный (глубокая иерархия, контекст, логирование)

Варианты 1-8 (Базовый уровень)
№	Предметная область	Базовое исключение	Производные исключения
1	Калькулятор	MathError	DivisionByZeroError, InvalidOperationError
2	Валидация email	EmailError	InvalidFormatError, EmptyEmailError
3	Работа с файлами	FileError	FileNotFoundError, AccessDeniedError
4	Пользователи	UserError	UserNotFoundError, DuplicateUserError
5	База данных	DBError	ConnectionError, QueryError
6	Сеть	NetworkError	TimeoutError, ConnectionRefusedError
7	Авторизация	AuthError	InvalidPasswordError, UserBlockedError
8	Конфигурация	ConfigError	MissingKeyError, InvalidValueError
Пример варианта 1:

python
class MathError(Exception):
    pass

class DivisionByZeroError(MathError):
    pass

class InvalidOperationError(MathError):
    pass

def divide(a, b):
    if b == 0:
        raise DivisionByZeroError("Деление на ноль!")
    return a / b
Варианты 9-17 (Средний уровень)
№	Предметная область	Особенности
9	Банковская система	Снятие, пополнение, перевод, лимиты
10	Интернет-магазин	Товары, корзина, заказы, оплата
11	Система бронирования	Отели, номера, даты, отмена
12	Социальная сеть	Друзья, посты, комментарии, лайки
13	Система тестирования	Вопросы, ответы, оценка, таймер
14	CRM-система	Клиенты, сделки, контакты, задачи
15	Система логистики	Доставка, склад, маршруты, грузы
16	Система голосования	Выборы, кандидаты, бюллетени
17	Система плагинов	Загрузка, активация, зависимости
Пример варианта 9:

python
class BankError(Exception):
    pass

class AccountError(BankError):
    pass

class InsufficientFundsError(AccountError):
    def __init__(self, amount, balance):
        self.amount = amount
        self.balance = balance
        super().__init__(f"Недостаточно средств: {amount} > {balance}")

class TransferError(BankError):
    pass

class SameAccountError(TransferError):
    pass
Варианты 18-25 (Сложный уровень)
№	Предметная область	Дополнительные требования
18	Компилятор	Лексический, синтаксический, семантический анализ
19	Игровой движок	Загрузка ресурсов, физика, рендеринг
20	AI/ML пайплайн	Данные, модель, обучение, инференс
21	Платформа IoT	Устройства, датчики, протоколы, данные
22	Блокчейн	Транзакции, консенсус, валидация
23	CI/CD система	Сборка, тесты, деплой, мониторинг
24	API-шлюз	Аутентификация, лимиты, маршрутизация
25	Оркестрация микросервисов	Сервисы, сеть, отказоустойчивость
Пример варианта 18:

python
class CompilerError(Exception):
    pass

class LexicalError(CompilerError):
    def __init__(self, line, col, char):
        self.line = line
        self.col = col
        self.char = char
        super().__init__(f"Лексическая ошибка [{line}:{col}]: неожиданный символ '{char}'")

class SyntaxError(CompilerError):
    def __init__(self, line, col, expected, found):
        super().__init__(f"Синтаксическая ошибка [{line}:{col}]: ожидалось {expected}, найдено {found}")

class SemanticError(CompilerError):
    pass
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Иерархия не создана или неправильно реализована
3 (удовлетворительно)	Созданы 3-4 исключения, есть базовый класс
4 (хорошо)	Создана полная иерархия (5-6 исключений), есть атрибуты
5 (отлично)	Глубокая иерархия, переопределены __init__ и __str__, есть примеры использования
5. Шпаргалка
5.1. Шаблон пользовательского исключения
python
class MyException(Exception):
    """Документация исключения."""
    
    def __init__(self, message: str, code: int = None, **kwargs):
        self.message = message
        self.code = code
        self.details = kwargs
        super().__init__(message)
    
    def __str__(self) -> str:
        if self.code:
            return f"[{self.code}] {self.message}"
        return self.message
5.2. Шаблон иерархии
python
# Базовое исключение
class DomainError(Exception):
    pass

# Категории ошибок
class CategoryAError(DomainError):
    pass

class CategoryBError(DomainError):
    pass

# Конкретные ошибки
class ConcreteError1(CategoryAError):
    pass

class ConcreteError2(CategoryAError):
    pass

class ConcreteError3(CategoryBError):
    pass
5.3. Перехват иерархии
python
try:
    risky_operation()
except ConcreteError1 as e:
    print(f"Конкретная ошибка 1: {e}")
except CategoryAError as e:
    print(f"Ошибка категории A: {e}")
except DomainError as e:
    print(f"Любая ошибка домена: {e}")
5.4. Полезные атрибуты исключений
python
class DetailedError(Exception):
    def __init__(self, message, **context):
        self.message = message
        self.context = context
        super().__init__(message)
    
    def __str__(self):
        if self.context:
            return f"{self.message} (контекст: {self.context})"
        return self.message
Карточка студента
text
ПЗ 2.15. СОЗДАНИЕ ИЕРАРХИИ ПОЛЬЗОВАТЕЛЬСКИХ ИСКЛЮЧЕНИЙ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Предметная область: _________________________________

=== ИЕРАРХИЯ ИСКЛЮЧЕНИЙ ===

Базовое исключение: _________________________________

Уровень 1 (категории):
1. _________________________________
2. _________________________________
3. _________________________________

Уровень 2 (конкретные):
1. _________________________________
2. _________________________________
3. _________________________________
4. _________________________________
5. _________________________________

=== АТРИБУТЫ ИСКЛЮЧЕНИЙ ===

□ message (сообщение)
□ code (код ошибки)
□ details (дополнительные данные)
□ Пользовательские атрибуты: _______________

=== ОТЧЁТ ===

Код иерархии прилагается
Примеры использования прилагаются
Демонстрация перехвата прилагается

Дата выполнения: _____________
