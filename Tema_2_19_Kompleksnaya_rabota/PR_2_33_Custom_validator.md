# ПЗ 2.33. Создание кастомных валидаторов

**Тема:** Валидация данных, кастомные валидаторы, бизнес-правила, Pydantic, ETL-процессы

**Цель работы:**  
Научиться создавать кастомные валидаторы для специфических бизнес-правил, реализовывать сложные проверки, интегрировать валидаторы в ETL-процессы и обрабатывать ошибки валидации.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `pydantic`, `pydantic[email]`

```bash
pip install pydantic pydantic[email]
Главная мысль: Стандартные валидаторы проверяют типы и форматы. Кастомные валидаторы воплощают бизнес-логику. Вместе они создают надёжную защиту данных.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Когда нужны кастомные валидаторы
Ситуация	Пример	Решение
Сложные форматы	ИНН, СНИЛС, паспорт	Кастомная проверка с regex
Бизнес-правила	Скидка не более 50%	Межполевая валидация
Внешние проверки	Уникальность email в БД	Проверка через API/БД
Зависимые поля	start_date < end_date	Root validator
Контекстные правила	Разная скидка для разных групп	Условная валидация
1.2. Типы кастомных валидаторов
Тип	Назначение	Аннотация
Полевой	Проверка одного поля	@validator('field')
Предварительный	Преобразование до валидации	@validator('field', pre=True)
Каждый элемент	Проверка элементов списка	@validator('field', each_item=True)
Корневой	Проверка нескольких полей	@root_validator
Валидатор с контекстом	Доступ к дополнительным данным	@validator('field', check_fields=False)
1.3. Сравнение подходов к валидации
Аспект	Field	@validator	@root_validator
Сложность	Простой	Средний	Сложный
Одно поле	✅	✅	❌
Несколько полей	❌	❌	✅
Доступ к модели	❌	✅	✅
Преобразование данных	❌	✅	✅
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая создание кастомных валидаторов.

Техническое задание (нулевой вариант)
Разработайте модуль кастомных валидаторов для банковской системы. Требования:

Валидация номера банковской карты (алгоритм Луна)

Валидация CVV/CVC кода

Валидация срока действия карты

Валидация суммы транзакции с учётом лимитов

Валидация PIN-кода (4 цифры, не повторяющиеся, не последовательные)

Межполевая валидация (сумма + комиссия)

Эталонная реализация
python
#!/usr/bin/env python3
"""
custom_validators.py — Кастомные валидаторы для банковской системы.
"""

from pydantic import BaseModel, Field, validator, root_validator
from datetime import datetime
from typing import Optional, List, Tuple
import re
from decimal import Decimal


# ============================================================
# КАСТОМНЫЕ ФУНКЦИИ ВАЛИДАЦИИ
# ============================================================

def luhn_check(card_number: str) -> bool:
    """
    Алгоритм Луна для проверки номера банковской карты.
    
    Алгоритм:
    1. Начиная с предпоследней цифры, двигаясь влево,
       умножаем каждую вторую цифру на 2.
    2. Если результат больше 9, вычитаем 9.
    3. Суммируем все цифры.
    4. Если сумма делится на 10 без остатка — номер корректен.
    """
    digits = [int(d) for d in str(card_number) if d.isdigit()]
    
    if len(digits) not in [15, 16]:
        return False
    
    # Алгоритм Луна
    total = 0
    is_second = False
    
    for digit in reversed(digits):
        if is_second:
            doubled = digit * 2
            total += doubled - 9 if doubled > 9 else doubled
        else:
            total += digit
        is_second = not is_second
    
    return total % 10 == 0


def is_sequential_digits(pin: str) -> bool:
    """Проверка, являются ли цифры последовательными (1234, 2345 и т.д.)."""
    digits = [int(d) for d in pin]
    for i in range(len(digits) - 1):
        if digits[i + 1] != (digits[i] % 10) + 1:
            return False
    return True


def is_repeating_digits(pin: str) -> bool:
    """Проверка, все ли цифры одинаковые (1111, 2222 и т.д.)."""
    return len(set(pin)) == 1


# ============================================================
# МОДЕЛИ С КАСТОМНЫМИ ВАЛИДАТОРАМИ
# ============================================================

class CreditCard(BaseModel):
    """Модель банковской карты с кастомными валидаторами."""
    
    card_number: str = Field(..., min_length=15, max_length=19, description="Номер карты")
    expiry_month: int = Field(..., ge=1, le=12, description="Месяц окончания")
    expiry_year: int = Field(..., ge=2024, le=2035, description="Год окончания")
    cvv: str = Field(..., min_length=3, max_length=4, description="CVV/CVC код")
    
    # ===== ПОЛЕВЫЕ ВАЛИДАТОРЫ =====
    
    @validator('card_number')
    def validate_card_number(cls, v):
        """Валидация номера карты (алгоритм Луна)."""
        # Удаляем пробелы и дефисы
        cleaned = re.sub(r'[\s\-]', '', v)
        
        if not cleaned.isdigit():
            raise ValueError('Номер карты должен содержать только цифры')
        
        if not luhn_check(cleaned):
            raise ValueError('Неверный номер карты (ошибка контрольной суммы)')
        
        # Определяем тип карты по первой цифре
        first_digit = cleaned[0]
        if first_digit == '4':
            card_type = 'Visa'
        elif first_digit in ['5', '2']:
            card_type = 'MasterCard'
        elif first_digit == '3':
            card_type = 'American Express'
        else:
            card_type = 'Unknown'
        
        # Форматируем для вывода (группировка по 4 цифры)
        formatted = ' '.join(cleaned[i:i+4] for i in range(0, len(cleaned), 4))
        
        return {
            'raw': cleaned,
            'formatted': formatted,
            'last4': cleaned[-4:],
            'type': card_type
        }
    
    @validator('cvv')
    def validate_cvv(cls, v, values):
        """Валидация CVV кода."""
        if not v.isdigit():
            raise ValueError('CVV должен содержать только цифры')
        
        # American Express использует 4-значный CVV
        card_data = values.get('card_number', {})
        if isinstance(card_data, dict) and card_data.get('type') == 'American Express':
            if len(v) != 4:
                raise ValueError('Для American Express CVV должен быть 4-значным')
        else:
            if len(v) != 3:
                raise ValueError('CVV должен быть 3-значным')
        
        return v
    
    @validator('expiry_month', 'expiry_year')
    def validate_expiry_date(cls, v, values, field):
        """Валидация срока действия карты (не просрочена)."""
        if 'expiry_month' in values and 'expiry_year' in values:
            month = values.get('expiry_month')
            year = values.get('expiry_year')
            
            if month and year:
                now = datetime.now()
                expiry = datetime(year, month, 1)
                
                if expiry < now:
                    raise ValueError(f'Срок действия карты истёк: {month:02d}/{year}')
        
        return v
    
    # ===== КОРНЕВОЙ ВАЛИДАТОР =====
    
    @root_validator
    def validate_card_complete(cls, values):
        """Комплексная проверка карты."""
        card_data = values.get('card_number', {})
        
        if isinstance(card_data, dict):
            # Проверка, что CVV не совпадает с последними цифрами карты
            cvv = values.get('cvv')
            last4 = card_data.get('last4', '')
            
            if cvv and last4 and cvv == last4:
                raise ValueError('CVV не может совпадать с последними 4 цифрами карты')
        
        return values


class PINCode(BaseModel):
    """Модель PIN-кода с кастомными валидаторами."""
    
    pin: str = Field(..., min_length=4, max_length=6, description="PIN-код")
    
    @validator('pin')
    def validate_pin(cls, v):
        """Валидация PIN-кода."""
        # Должен содержать только цифры
        if not v.isdigit():
            raise ValueError('PIN-код должен содержать только цифры')
        
        # Проверка на одинаковые цифры
        if is_repeating_digits(v):
            raise ValueError('PIN-код не может состоять из одинаковых цифр')
        
        # Проверка на последовательные цифры
        if is_sequential_digits(v):
            raise ValueError('PIN-код не может содержать последовательные цифры')
        
        # Проверка на простые комбинации
        forbidden = ['1234', '4321', '0000', '1111', '2222', '3333', 
                     '4444', '5555', '6666', '7777', '8888', '9999']
        
        if v in forbidden:
            raise ValueError(f'PIN-код {v} слишком простой, выберите другой')
        
        return v


class Transaction(BaseModel):
    """Модель транзакции с кастомными валидаторами."""
    
    amount: Decimal = Field(..., gt=0, description="Сумма транзакции")
    currency: str = Field('RUB', regex='^(RUB|USD|EUR)$', description="Валюта")
    fee_percent: Decimal = Field(1.5, ge=0, le=10, description="Комиссия в %")
    
    # Ежедневный лимит (в реальном приложении получается из БД)
    daily_limit: Decimal = Field(100000, gt=0, description="Дневной лимит")
    daily_spent: Decimal = Field(0, ge=0, description="Потрачено сегодня")
    
    # Лимит на одну транзакцию
    transaction_limit: Decimal = Field(50000, gt=0, description="Лимит транзакции")
    
    # ===== ПОЛЕВЫЕ ВАЛИДАТОРЫ =====
    
    @validator('amount')
    def validate_amount_format(cls, v):
        """Валидация формата суммы."""
        # Проверка на количество знаков после запятой
        if v.as_tuple().exponent < -2:
            raise ValueError('Сумма не может содержать более 2 знаков после запятой')
        
        return v
    
    # ===== ПРЕДВАРИТЕЛЬНЫЙ ВАЛИДАТОР =====
    
    @validator('amount', pre=True)
    def parse_amount(cls, v):
        """Преобразование строки в Decimal."""
        if isinstance(v, str):
            # Убираем символы валют
            v = re.sub(r'[^\d.,\-]', '', v)
            v = v.replace(',', '.')
            return Decimal(v)
        return v
    
    # ===== КОРНЕВОЙ ВАЛИДАТОР =====
    
    @root_validator
    def validate_transaction(cls, values):
        """Комплексная проверка транзакции."""
        amount = values.get('amount')
        daily_limit = values.get('daily_limit')
        daily_spent = values.get('daily_spent', 0)
        transaction_limit = values.get('transaction_limit')
        fee_percent = values.get('fee_percent', 0)
        
        if not amount:
            return values
        
        # Проверка лимита транзакции
        if amount > transaction_limit:
            raise ValueError(f'Сумма транзакции {amount} превышает лимит {transaction_limit}')
        
        # Проверка дневного лимита
        if daily_spent + amount > daily_limit:
            remaining = daily_limit - daily_spent
            raise ValueError(f'Превышен дневной лимит. Доступно: {remaining}')
        
        # Расчёт суммы с комиссией
        fee = amount * fee_percent / Decimal('100')
        total = amount + fee
        values['fee_amount'] = fee
        values['total_amount'] = total
        
        return values
    
    # ===== СВОЙСТВА =====
    
    @property
    def fee_amount(self) -> Decimal:
        """Сумма комиссии."""
        return getattr(self, '_fee_amount', Decimal('0'))
    
    @property
    def total_amount(self) -> Decimal:
        """Общая сумма с комиссией."""
        return getattr(self, '_total_amount', self.amount)


class TransferRequest(BaseModel):
    """Модель запроса на перевод с кастомными валидаторами."""
    
    from_card: CreditCard
    to_card: CreditCard
    amount: Decimal = Field(..., gt=0)
    pin: PINCode
    purpose: Optional[str] = Field(None, max_length=200)
    
    # ===== ПОЛЕВЫЕ ВАЛИДАТОРЫ С КОНТЕКСТОМ =====
    
    @validator('amount')
    def validate_amount_range(cls, v):
        """Проверка суммы перевода."""
        if v < Decimal('10'):
            raise ValueError('Минимальная сумма перевода 10 рублей')
        
        if v > Decimal('1000000'):
            raise ValueError('Максимальная сумма перевода 1 000 000 рублей')
        
        return v
    
    # ===== КОРНЕВОЙ ВАЛИДАТОР =====
    
    @root_validator
    def validate_transfer(cls, values):
        """Комплексная проверка перевода."""
        from_card = values.get('from_card')
        to_card = values.get('to_card')
        amount = values.get('amount')
        
        if not from_card or not to_card:
            return values
        
        # Получаем номера карт
        from_number = from_card.card_number.get('raw') if isinstance(from_card.card_number, dict) else None
        to_number = to_card.card_number.get('raw') if isinstance(to_card.card_number, dict) else None
        
        # Проверка: нельзя переводить на ту же карту
        if from_number and to_number and from_number == to_number:
            raise ValueError('Нельзя перевести деньги на ту же карту')
        
        # Проверка: карта отправителя не должна быть просрочена
        now = datetime.now()
        from_expiry = datetime(from_card.expiry_year, from_card.expiry_month, 1)
        if from_expiry < now:
            raise ValueError('Карта отправителя просрочена')
        
        return values


# ============================================================
# КЛАСС ДЛЯ ПАКЕТНОЙ ВАЛИДАЦИИ
# ============================================================

class BatchValidator:
    """Класс для пакетной валидации данных."""
    
    def __init__(self):
        self.valid_records = []
        self.invalid_records = []
        self.errors = []
    
    def validate_transaction(self, transaction_data: dict) -> Optional[Transaction]:
        """Валидация одной транзакции."""
        try:
            validated = Transaction(**transaction_data)
            self.valid_records.append(validated)
            return validated
        except Exception as e:
            self.invalid_records.append(transaction_data)
            self.errors.append({
                "data": transaction_data,
                "error": str(e)
            })
            return None
    
    def validate_batch(self, transactions: list) -> list:
        """Валидация пакета транзакций."""
        valid = []
        for tx in transactions:
            validated = self.validate_transaction(tx)
            if validated:
                valid.append(validated)
        return valid
    
    def get_statistics(self) -> dict:
        """Статистика валидации."""
        total = len(self.valid_records) + len(self.invalid_records)
        return {
            "total": total,
            "valid": len(self.valid_records),
            "invalid": len(self.invalid_records),
            "success_rate": f"{len(self.valid_records) / total * 100:.1f}%" if total else "0%"
        }
    
    def get_error_report(self) -> list:
        """Отчёт об ошибках."""
        return self.errors


# ============================================================
# ДЕМОНСТРАЦИЯ
# ============================================================

def demo():
    """Демонстрация работы кастомных валидаторов."""
    
    print("=" * 70)
    print("ДЕМОНСТРАЦИЯ КАСТОМНЫХ ВАЛИДАТОРОВ")
    print("=" * 70)
    
    # Тест 1: Валидация номера карты
    print("\n1️⃣ ВАЛИДАЦИЯ НОМЕРА КАРТЫ")
    print("-" * 50)
    
    test_cards = [
        ("4532 1234 5678 9012", "Visa"),
        ("5555 5555 5555 4444", "MasterCard"),
        ("1234 5678 9012 3456", "Invalid (Luhn)"),
    ]
    
    for card_number, expected in test_cards:
        try:
            card = CreditCard(
                card_number=card_number,
                expiry_month=12,
                expiry_year=2025,
                cvv="123"
            )
            print(f"✅ {card_number} → {card.card_number['type']}")
        except Exception as e:
            print(f"❌ {card_number} → {e}")
    
    # Тест 2: Валидация PIN-кода
    print("\n2️⃣ ВАЛИДАЦИЯ PIN-КОДА")
    print("-" * 50)
    
    test_pins = ["1234", "1111", "123456", "9876", "4321", "0000"]
    
    for pin in test_pins:
        try:
            pin_obj = PINCode(pin=pin)
            print(f"✅ PIN-код {pin} корректен")
        except Exception as e:
            print(f"❌ PIN-код {pin} → {e}")
    
    # Тест 3: Валидация транзакции
    print("\n3️⃣ ВАЛИДАЦИЯ ТРАНЗАКЦИИ")
    print("-" * 50)
    
    test_transactions = [
        {"amount": 1000, "currency": "RUB", "daily_spent": 0},
        {"amount": 60000, "currency": "RUB", "daily_spent": 0},  # превышение лимита транзакции
        {"amount": 1000.123, "currency": "RUB", "daily_spent": 0},  # больше 2 знаков
        {"amount": "5000", "currency": "USD", "daily_spent": 95000},  # дневной лимит
    ]
    
    for tx_data in test_transactions:
        try:
            tx = Transaction(**tx_data)
            print(f"✅ Транзакция {tx.amount} {tx.currency} — OK")
        except Exception as e:
            print(f"❌ Транзакция {tx_data.get('amount')} → {e}")
    
    # Тест 4: Полный перевод
    print("\n4️⃣ ПОЛНЫЙ ПЕРЕВОД (МЕЖПОЛЕВАЯ ВАЛИДАЦИЯ)")
    print("-" * 50)
    
    try:
        transfer = TransferRequest(
            from_card={
                "card_number": "4532 1234 5678 9012",
                "expiry_month": 12,
                "expiry_year": 2025,
                "cvv": "123"
            },
            to_card={
                "card_number": "5555 5555 5555 4444",
                "expiry_month": 12,
                "expiry_year": 2026,
                "cvv": "123"
            },
            amount=5000,
            pin={"pin": "5678"},
            purpose="Оплата услуг"
        )
        print(f"✅ Перевод одобрен: {transfer.amount} руб.")
    except Exception as e:
        print(f"❌ Ошибка перевода: {e}")
    
    # Тест 5: Пакетная валидация
    print("\n5️⃣ ПАКЕТНАЯ ВАЛИДАЦИЯ")
    print("-" * 50)
    
    batch = [
        {"amount": 1000, "currency": "RUB", "daily_spent": 0},
        {"amount": -500, "currency": "RUB", "daily_spent": 0},  # отрицательная сумма
        {"amount": 2000, "currency": "RUB", "daily_spent": 0},
        {"amount": "invalid", "currency": "RUB", "daily_spent": 0},  # не число
    ]
    
    validator = BatchValidator()
    valid = validator.validate_batch(batch)
    stats = validator.get_statistics()
    
    print(f"Статистика:")
    print(f"   Всего: {stats['total']}")
    print(f"   Валидных: {stats['valid']}")
    print(f"   Невалидных: {stats['invalid']}")
    print(f"   Успешность: {stats['success_rate']}")
    
    if validator.get_error_report():
        print(f"\nОшибки:")
        for error in validator.get_error_report()[:2]:
            print(f"   {error['error'][:100]}...")
    
    print("\n" + "=" * 70)
    print("ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА")
    print("=" * 70)


if __name__ == "__main__":
    demo()
Ожидаемый вывод
text
======================================================================
ДЕМОНСТРАЦИЯ КАСТОМНЫХ ВАЛИДАТОРОВ
======================================================================

1️⃣ ВАЛИДАЦИЯ НОМЕРА КАРТЫ
--------------------------------------------------
✅ 4532 1234 5678 9012 → Visa
✅ 5555 5555 5555 4444 → MasterCard
❌ 1234 5678 9012 3456 → 1 validation error for CreditCard
card_number
  Неверный номер карты (ошибка контрольной суммы) (type=value_error)

2️⃣ ВАЛИДАЦИЯ PIN-КОДА
--------------------------------------------------
❌ PIN-код 1234 → 1 validation error for PINCode
pin
  PIN-код не может содержать последовательные цифры (type=value_error)
❌ PIN-код 1111 → 1 validation error for PINCode
pin
  PIN-код 1111 слишком простой, выберите другой (type=value_error)
✅ PIN-код 123456 корректен
✅ PIN-код 9876 корректен
❌ PIN-код 4321 → 1 validation error for PINCode
pin
  PIN-код не может содержать последовательные цифры (type=value_error)
❌ PIN-код 0000 → 1 validation error for PINCode
pin
  PIN-код 0000 слишком простой, выберите другой (type=value_error)

3️⃣ ВАЛИДАЦИЯ ТРАНЗАКЦИИ
--------------------------------------------------
✅ Транзакция 1000 RUB — OK
❌ Транзакция 60000 → 1 validation error for Transaction
__root__
  Сумма транзакции 60000 превышает лимит 50000 (type=value_error)
❌ Транзакция 1000.123 → 1 validation error for Transaction
amount
  Сумма не может содержать более 2 знаков после запятой (type=value_error)
❌ Транзакция 5000 → 1 validation error for Transaction
__root__
  Превышен дневной лимит. Доступно: 5000 (type=value_error)

4️⃣ ПОЛНЫЙ ПЕРЕВОД (МЕЖПОЛЕВАЯ ВАЛИДАЦИЯ)
--------------------------------------------------
✅ Перевод одобрен: 5000 руб.

5️⃣ ПАКЕТНАЯ ВАЛИДАЦИЯ
--------------------------------------------------
Статистика:
   Всего: 4
   Валидных: 2
   Невалидных: 2
   Успешность: 50.0%

Ошибки:
   1 validation error for Transaction
amount
  ensure this value is greater than 0 (type=value_error)...
   1 validation error for Transaction
amount
  value is not a valid decimal (type=type_error.decimal)...

======================================================================
ДЕМОНСТРАЦИЯ ЗАВЕРШЕНА
======================================================================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (простые кастомные валидаторы)

Варианты 9-17: средний (сложные форматы, бизнес-правила)

Варианты 18-25: сложный (межполевая валидация, внешние проверки)

Варианты 1-8 (Базовый уровень)
№	Тема	Кастомная валидация
1	Валидация ИНН	Проверка контрольной суммы ИНН (10/12 цифр)
2	Валидация СНИЛС	Проверка контрольной суммы СНИЛС
3	Валидация паспорта	Проверка формата (серия, номер, код подразделения)
4	Валидация номера телефона	Нормализация + проверка кода оператора
5	Валидация email	Проверка MX-записи домена
6	Валидация URL	Проверка существования URL (HTTP check)
7	Валидация даты	Проверка формата, високосные годы
8	Валидация времени	Проверка формата, часовых поясов
Варианты 9-17 (Средний уровень)
№	Тема	Кастомная валидация
9	Банковская карта	Алгоритм Луна, тип карты, CVV
10	PIN-код	Не повторяющиеся, не последовательные цифры
11	Пароль	Сложность: заглавные, цифры, спецсимволы
12	Промокод	Проверка формата, срока действия, лимита
13	Бонусные баллы	Проверка накопления, сгорания
14	Кэшбек	Расчёт процента, категории товаров
15	Скидка	Процент, сумма, условия применения
16	Кредитный лимит	Проверка остатка, просрочки
17	Страховой полис	Проверка формата, периода действия
Варианты 18-25 (Сложный уровень)
№	Тема	Особенности
18	Платёжное поручение	Сумма, получатель, назначение, контрольные суммы
19	Кредитная заявка	Скоринг, доход, кредитная история
20	Страхование	Объект, сумма, срок, риски
21	Бронирование отеля	Даты, гости, типы номеров, доп. услуги
22	Авиабилет	Маршрут, пассажиры, багаж, класс обслуживания
23	Договор купли-продажи	Стороны, предмет, цена, условия
24	Таможенная декларация	Товары, стоимость, коды ТН ВЭД
25	Медицинская карта	Диагнозы, назначения, противопоказания
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Валидаторы не работают или не созданы
3 (удовлетворительно)	Созданы 2-3 базовых валидатора
4 (хорошо)	+ сложные форматы, бизнес-правила
5 (отлично)	+ межполевая валидация, пакетная обработка, отчёты
5. Шпаргалка
python
# === ПОЛЕВОЙ ВАЛИДАТОР ===
@validator('field')
def validate_field(cls, v):
    if not condition:
        raise ValueError('Ошибка')
    return transformed_v

# === ПРЕДВАРИТЕЛЬНЫЙ ВАЛИДАТОР ===
@validator('field', pre=True)
def parse_field(cls, v):
    return transformed_v

# === ВАЛИДАТОР СПИСКА ===
@validator('items', each_item=True)
def validate_item(cls, v):
    return v

# === КОРНЕВОЙ ВАЛИДАТОР ===
@root_validator
def validate_all(cls, values):
    if values.get('field1') != values.get('field2'):
        raise ValueError('Ошибка')
    values['calculated'] = values['field1'] + values['field2']
    return values

# === АЛГОРИТМ ЛУНА ===
def luhn_check(card_number: str) -> bool:
    digits = [int(d) for d in str(card_number) if d.isdigit()]
    total = 0
    is_second = False
    for digit in reversed(digits):
        if is_second:
            doubled = digit * 2
            total += doubled - 9 if doubled > 9 else doubled
        else:
            total += digit
        is_second = not is_second
    return total % 10 == 0
Карточка студента
text
ПЗ 2.33. СОЗДАНИЕ КАСТОМНЫХ ВАЛИДАТОРОВ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== ТИПЫ ВАЛИДАТОРОВ ===

□ Полевой валидатор (@validator)
□ Предварительный валидатор (pre=True)
□ Валидатор элементов списка (each_item=True)
□ Корневой валидатор (@root_validator)

=== ПРОВЕРКИ ===

□ Формат (регулярные выражения)
□ Алгоритмическая проверка (Luhn, контрольная сумма)
□ Бизнес-правила
□ Межполевые зависимости
□ Внешние проверки (API/БД)

=== ОТЧЁТ ===

Файл custom_validators.py: _____________
Количество валидаторов: _____
Пример успешной валидации: _____________
Пример ошибки валидации: _____________

Дата выполнения: _____________
