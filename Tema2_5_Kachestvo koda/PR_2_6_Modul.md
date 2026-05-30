# ПЗ 2.6. Разработка модуля для расчёта временных интервалов и преобразования часовых поясов

**Тема:** Работа с датой и временем в Python (datetime, timezone, timedelta, pytz)

**Цель работы:**  
Научиться разрабатывать модули для профессиональной работы с датами, временем и часовыми поясами: вычисление интервалов, преобразование между поясами, форматирование, парсинг и валидация.

**Время выполнения:** 90-120 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.9+ (рекомендуется 3.10+ для zoneinfo)
- Установленные пакеты:

```bash
pip install pytz python-dateutil
Нулевой вариант (эталонный) — Демонстрация преподавателя
Назначение: преподаватель выполняет этот вариант на занятии, показывая разработку модуля для работы с датами и часовыми поясами.

Техническое задание (нулевой вариант)
Разработайте модуль datetime_utils.py, который предоставляет следующий функционал:

Конвертация между часовыми поясами

Расчёт временных интервалов (рабочие дни, часы)

Форматирование дат в человеко-читаемый вид

Парсинг дат из разных форматов

Получение начала и конца дня/недели/месяца

Эталонная реализация
python
#!/usr/bin/env python3
"""
Модуль datetime_utils.py

Утилиты для работы с датами, временем и часовыми поясами.

Примеры использования:
    >>> from datetime_utils import DateTimeUtils
    >>> utils = DateTimeUtils()
    >>> now = utils.now_in_timezone('Europe/Moscow')
    >>> print(utils.format_human(now))
    15 января 2024 года, 14:30
"""

from datetime import datetime, date, time, timedelta, timezone
from typing import Optional, Union, List, Tuple
import pytz
from dateutil import parser
import calendar


class DateTimeUtils:
    """
    Утилиты для работы с датами и временем.
    
    Поддерживает работу с часовыми поясами через pytz,
    форматирование, парсинг и вычисление интервалов.
    """
    
    # Константы для форматирования
    DATE_FORMAT_RU = "%d.%m.%Y"
    DATETIME_FORMAT_RU = "%d.%m.%Y %H:%M:%S"
    DATE_FORMAT_ISO = "%Y-%m-%d"
    DATETIME_FORMAT_ISO = "%Y-%m-%d %H:%M:%S"
    TIME_FORMAT = "%H:%M:%S"
    
    # Названия месяцев на русском
    MONTHS_RU = [
        "января", "февраля", "марта", "апреля", "мая", "июня",
        "июля", "августа", "сентября", "октября", "ноября", "декабря"
    ]
    
    # Названия дней недели на русском
    WEEKDAYS_RU = [
        "понедельник", "вторник", "среда", "четверг",
        "пятница", "суббота", "воскресенье"
    ]
    
    def __init__(self, default_timezone: str = "UTC"):
        """
        Инициализирует утилиту с часовым поясом по умолчанию.
        
        Args:
            default_timezone: Часовой пояс по умолчанию (например, 'Europe/Moscow')
        """
        self.default_timezone = pytz.timezone(default_timezone)
    
    # ==================== ЧАСОВЫЕ ПОЯСА ====================
    
    def now_in_timezone(self, timezone_name: Optional[str] = None) -> datetime:
        """
        Возвращает текущее время в указанном часовом поясе.
        
        Args:
            timezone_name: Название часового пояса (например, 'America/New_York').
                          Если не указан, используется пояс по умолчанию.
        
        Returns:
            Осведомлённый datetime в указанном часовом поясе
        """
        tz = pytz.timezone(timezone_name) if timezone_name else self.default_timezone
        return datetime.now(tz)
    
    def convert_timezone(
        self, 
        dt: datetime, 
        target_timezone: str,
        source_timezone: Optional[str] = None
    ) -> datetime:
        """
        Конвертирует datetime из одного часового пояса в другой.
        
        Args:
            dt: Исходная дата/время
            target_timezone: Целевой часовой пояс
            source_timezone: Исходный часовой пояс (если dt наивный)
        
        Returns:
            Конвертированная дата/время
        """
        # Если datetime наивный, добавляем исходный часовой пояс
        if dt.tzinfo is None:
            if source_timezone is None:
                source_timezone = "UTC"
            source_tz = pytz.timezone(source_timezone)
            dt = source_tz.localize(dt)
        
        # Конвертируем в целевой пояс
        target_tz = pytz.timezone(target_timezone)
        return dt.astimezone(target_tz)
    
    def get_utc_offset(self, timezone_name: str, dt: Optional[datetime] = None) -> str:
        """
        Возвращает смещение UTC для часового пояса.
        
        Args:
            timezone_name: Название часового пояса
            dt: Дата (для учёта летнего времени)
        
        Returns:
            Строка смещения (например, '+03:00')
        """
        tz = pytz.timezone(timezone_name)
        if dt is None:
            dt = datetime.now()
        
        if dt.tzinfo is None:
            dt = tz.localize(dt)
        
        offset = dt.utcoffset()
        if offset:
            hours = offset.seconds // 3600
            minutes = (offset.seconds % 3600) // 60
            sign = '+' if offset.days >= 0 else '-'
            return f"{sign}{hours:02d}:{minutes:02d}"
        return "+00:00"
    
    # ==================== ВРЕМЕННЫЕ ИНТЕРВАЛЫ ====================
    
    def time_diff(self, dt1: datetime, dt2: datetime) -> timedelta:
        """
        Вычисляет разницу между двумя датами.
        
        Args:
            dt1: Первая дата
            dt2: Вторая дата
        
        Returns:
            timedelta объект с разницей
        """
        # Приводим к общему часовому поясу если нужно
        if dt1.tzinfo is None and dt2.tzinfo is not None:
            dt1 = dt1.replace(tzinfo=dt2.tzinfo)
        elif dt1.tzinfo is not None and dt2.tzinfo is None:
            dt2 = dt2.replace(tzinfo=dt1.tzinfo)
        
        return dt2 - dt1 if dt2 > dt1 else dt1 - dt2
    
    def format_timedelta(self, delta: timedelta, granularity: str = "auto") -> str:
        """
        Форматирует timedelta в человеко-читаемый вид.
        
        Args:
            delta: timedelta объект
            granularity: Детализация ('auto', 'days', 'hours', 'minutes')
        
        Returns:
            Отформатированная строка (например, "5 дней 3 часа 15 минут")
        """
        days = delta.days
        hours = delta.seconds // 3600
        minutes = (delta.seconds % 3600) // 60
        seconds = delta.seconds % 60
        
        parts = []
        
        if granularity == "days":
            return f"{days} дней"
        elif granularity == "hours":
            total_hours = days * 24 + hours
            return f"{total_hours} часов"
        elif granularity == "minutes":
            total_minutes = days * 24 * 60 + hours * 60 + minutes
            return f"{total_minutes} минут"
        
        # auto: показываем только значимые единицы
        if days:
            parts.append(self._pluralize(days, "день", "дня", "дней"))
        if hours:
            parts.append(self._pluralize(hours, "час", "часа", "часов"))
        if minutes:
            parts.append(self._pluralize(minutes, "минута", "минуты", "минут"))
        if seconds and not parts:
            parts.append(self._pluralize(seconds, "секунда", "секунды", "секунд"))
        
        return " ".join(parts)
    
    def _pluralize(self, n: int, form1: str, form2: str, form5: str) -> str:
        """Возвращает существительное с правильным окончанием."""
        if 10 <= n % 100 <= 20:
            return f"{n} {form5}"
        elif n % 10 == 1:
            return f"{n} {form1}"
        elif 2 <= n % 10 <= 4:
            return f"{n} {form2}"
        else:
            return f"{n} {form5}"
    
    def business_days_between(self, start: date, end: date) -> int:
        """
        Вычисляет количество рабочих дней (пн-пт) между датами.
        
        Args:
            start: Начальная дата
            end: Конечная дата
        
        Returns:
            Количество рабочих дней
        """
        if start > end:
            start, end = end, start
        
        current = start
        business_days = 0
        
        while current <= end:
            if current.weekday() < 5:  # 0-4 = пн-пт
                business_days += 1
            current += timedelta(days=1)
        
        return business_days
    
    def add_business_days(self, start_date: date, days: int) -> date:
        """
        Добавляет указанное количество рабочих дней к дате.
        
        Args:
            start_date: Начальная дата
            days: Количество рабочих дней
        
        Returns:
            Новая дата
        """
        current = start_date
        added = 0
        
        while added < days:
            current += timedelta(days=1)
            if current.weekday() < 5:
                added += 1
        
        return current
    
    # ==================== ФОРМАТИРОВАНИЕ ====================
    
    def format_human(self, dt: datetime, lang: str = "ru") -> str:
        """
        Форматирует дату в человеко-читаемый вид.
        
        Args:
            dt: Дата/время
            lang: Язык ('ru' или 'en')
        
        Returns:
            Форматированная строка
        """
        if lang == "ru":
            month = self.MONTHS_RU[dt.month - 1]
            return f"{dt.day} {month} {dt.year} года, {dt.hour:02d}:{dt.minute:02d}"
        else:
            return dt.strftime("%B %d, %Y at %I:%M %p")
    
    def format_short(self, dt: datetime) -> str:
        """Краткий формат даты."""
        return dt.strftime(self.DATE_FORMAT_RU)
    
    def format_datetime(self, dt: datetime, iso: bool = False) -> str:
        """
        Стандартное форматирование даты-времени.
        
        Args:
            dt: Дата/время
            iso: Использовать ISO формат
        
        Returns:
            Отформатированная строка
        """
        if iso:
            return dt.isoformat()
        return dt.strftime(self.DATETIME_FORMAT_RU)
    
    # ==================== ПАРСИНГ ====================
    
    def parse_date(self, date_string: str, fuzzy: bool = False) -> Optional[datetime]:
        """
        Парсит дату из строки в разных форматах.
        
        Args:
            date_string: Строка с датой
            fuzzy: Разрешить нечёткий поиск
        
        Returns:
            datetime объект или None при ошибке
        """
        formats = [
            "%Y-%m-%d",
            "%d.%m.%Y",
            "%d/%m/%Y",
            "%d-%m-%Y",
            "%Y%m%d",
            "%Y-%m-%d %H:%M:%S",
            "%d.%m.%Y %H:%M:%S",
            "%d.%m.%Y %H:%M",
        ]
        
        for fmt in formats:
            try:
                return datetime.strptime(date_string, fmt)
            except ValueError:
                continue
        
        # Попытка через dateutil (более гибкий)
        try:
            return parser.parse(date_string, fuzzy=fuzzy)
        except (ValueError, OverflowError):
            return None
    
    def parse_datetime_with_timezone(self, date_string: str) -> Optional[datetime]:
        """
        Парсит строку с датой и часовым поясом.
        
        Args:
            date_string: Строка вида "2024-01-15 14:30:45+03:00"
        
        Returns:
            Осведомлённый datetime объект
        """
        try:
            return datetime.fromisoformat(date_string)
        except ValueError:
            return self.parse_date(date_string)
    
    # ==================== НАЧАЛО/КОНЕЦ ПЕРИОДОВ ====================
    
    def start_of_day(self, dt: datetime) -> datetime:
        """Возвращает начало дня (00:00:00)."""
        return datetime(dt.year, dt.month, dt.day)
    
    def end_of_day(self, dt: datetime) -> datetime:
        """Возвращает конец дня (23:59:59.999999)."""
        return datetime(dt.year, dt.month, dt.day, 23, 59, 59, 999999)
    
    def start_of_week(self, dt: datetime, week_start: int = 0) -> datetime:
        """
        Возвращает начало недели.
        
        Args:
            dt: Дата
            week_start: День начала недели (0=понедельник, 6=воскресенье)
        
        Returns:
            Дата начала недели
        """
        days_to_subtract = (dt.weekday() - week_start) % 7
        return self.start_of_day(dt - timedelta(days=days_to_subtract))
    
    def end_of_week(self, dt: datetime, week_start: int = 0) -> datetime:
        """Возвращает конец недели."""
        start = self.start_of_week(dt, week_start)
        return self.end_of_day(start + timedelta(days=6))
    
    def start_of_month(self, dt: datetime) -> datetime:
        """Возвращает начало месяца."""
        return datetime(dt.year, dt.month, 1)
    
    def end_of_month(self, dt: datetime) -> datetime:
        """Возвращает конец месяца."""
        last_day = calendar.monthrange(dt.year, dt.month)[1]
        return datetime(dt.year, dt.month, last_day, 23, 59, 59, 999999)
    
    def start_of_year(self, dt: datetime) -> datetime:
        """Возвращает начало года."""
        return datetime(dt.year, 1, 1)
    
    def end_of_year(self, dt: datetime) -> datetime:
        """Возвращает конец года."""
        return datetime(dt.year, 12, 31, 23, 59, 59, 999999)
    
    # ==================== ВАЛИДАЦИЯ ====================
    
    def is_valid_date(self, year: int, month: int, day: int) -> bool:
        """Проверяет, существует ли дата."""
        try:
            date(year, month, day)
            return True
        except ValueError:
            return False
    
    def is_weekend(self, dt: date) -> bool:
        """Проверяет, является ли день выходным."""
        return dt.weekday() >= 5
    
    def is_leap_year(self, year: int) -> bool:
        """Проверяет, является ли год високосным."""
        return calendar.isleap(year)
    
    def days_in_month(self, year: int, month: int) -> int:
        """Возвращает количество дней в месяце."""
        return calendar.monthrange(year, month)[1]


# ==================== ДОПОЛНИТЕЛЬНЫЕ УТИЛИТЫ ====================

class Timer:
    """Простой таймер для измерения интервалов."""
    
    def __init__(self):
        self.start_time = None
        self.end_time = None
    
    def start(self) -> None:
        """Запускает таймер."""
        self.start_time = datetime.now()
        self.end_time = None
    
    def stop(self) -> Optional[timedelta]:
        """Останавливает таймер и возвращает интервал."""
        if self.start_time is None:
            return None
        self.end_time = datetime.now()
        return self.end_time - self.start_time
    
    def elapsed(self) -> Optional[timedelta]:
        """Возвращает прошедшее время без остановки."""
        if self.start_time is None:
            return None
        return datetime.now() - self.start_time
    
    def reset(self) -> None:
        """Сбрасывает таймер."""
        self.start_time = None
        self.end_time = None


def age(birth_date: date) -> int:
    """
    Вычисляет возраст в годах.
    
    Args:
        birth_date: Дата рождения
    
    Returns:
        Количество полных лет
    """
    today = date.today()
    age = today.year - birth_date.year
    if (today.month, today.day) < (birth_date.month, birth_date.day):
        age -= 1
    return age


def days_until(target_date: date) -> int:
    """
    Вычисляет количество дней до указанной даты.
    
    Args:
        target_date: Целевая дата
    
    Returns:
        Количество дней
    """
    today = date.today()
    return (target_date - today).days


# ==================== ПРИМЕР ИСПОЛЬЗОВАНИЯ ====================

def main():
    """Демонстрация работы модуля."""
    utils = DateTimeUtils(default_timezone="Europe/Moscow")
    
    print("=" * 60)
    print("ДЕМОНСТРАЦИЯ РАБОТЫ МОДУЛЯ datetime_utils")
    print("=" * 60)
    
    # 1. Текущее время в разных часовых поясах
    print("\n1. ЧАСОВЫЕ ПОЯСА:")
    print(f"   Москва: {utils.now_in_timezone('Europe/Moscow')}")
    print(f"   Нью-Йорк: {utils.now_in_timezone('America/New_York')}")
    print(f"   Токио: {utils.now_in_timezone('Asia/Tokyo')}")
    print(f"   Смещение MSK: {utils.get_utc_offset('Europe/Moscow')}")
    
    # 2. Конвертация времени
    print("\n2. КОНВЕРТАЦИЯ:")
    moscow_time = utils.now_in_timezone('Europe/Moscow')
    ny_time = utils.convert_timezone(moscow_time, 'America/New_York')
    print(f"   Москва: {moscow_time}")
    print(f"   Нью-Йорк: {ny_time}")
    
    # 3. Временные интервалы
    print("\n3. ВРЕМЕННЫЕ ИНТЕРВАЛЫ:")
    start = datetime(2024, 1, 1)
    end = datetime(2024, 12, 31)
    delta = utils.time_diff(start, end)
    print(f"   Между {start.date()} и {end.date()}:")
    print(f"   {utils.format_timedelta(delta)}")
    
    # 4. Рабочие дни
    print("\n4. РАБОЧИЕ ДНИ:")
    start_date = date(2024, 1, 1)
    end_date = date(2024, 12, 31)
    business_days = utils.business_days_between(start_date, end_date)
    print(f"   Рабочих дней в 2024 году: {business_days}")
    
    next_friday = utils.add_business_days(date.today(), 5)
    print(f"   Через 5 рабочих дней: {next_friday}")
    
    # 5. Форматирование
    print("\n5. ФОРМАТИРОВАНИЕ:")
    now = datetime.now()
    print(f"   Человеческий формат: {utils.format_human(now)}")
    print(f"   Краткий формат: {utils.format_short(now)}")
    print(f"   ISO: {utils.format_datetime(now, iso=True)}")
    
    # 6. Парсинг
    print("\n6. ПАРСИНГ:")
    parsed = utils.parse_date("15.01.2024")
    print(f"   '15.01.2024' → {parsed}")
    parsed = utils.parse_date("2024-01-15 14:30:00")
    print(f"   '2024-01-15 14:30:00' → {parsed}")
    
    # 7. Начало/конец периодов
    print("\n7. ГРАНИЦЫ ПЕРИОДОВ:")
    some_date = datetime(2024, 6, 15, 14, 30)
    print(f"   Исходная дата: {some_date}")
    print(f"   Начало дня: {utils.start_of_day(some_date)}")
    print(f"   Начало недели: {utils.start_of_week(some_date)}")
    print(f"   Начало месяца: {utils.start_of_month(some_date)}")
    print(f"   Начало года: {utils.start_of_year(some_date)}")
    
    # 8. Валидация
    print("\n8. ВАЛИДАЦИЯ:")
    print(f"   29.02.2024 существует? {utils.is_valid_date(2024, 2, 29)}")
    print(f"   29.02.2023 существует? {utils.is_valid_date(2023, 2, 29)}")
    print(f"   2024 високосный? {utils.is_leap_year(2024)}")
    print(f"   Дней в феврале 2024: {utils.days_in_month(2024, 2)}")
    
    # 9. Таймер
    print("\n9. ТАЙМЕР:")
    timer = Timer()
    timer.start()
    import time
    time.sleep(0.5)
    print(f"   Прошло: {timer.elapsed()}")
    timer.stop()
    
    print("\n" + "=" * 60)
    print("Демонстрация завершена")
    print("=" * 60)


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
============================================================
ДЕМОНСТРАЦИЯ РАБОТЫ МОДУЛЯ datetime_utils
============================================================

1. ЧАСОВЫЕ ПОЯСА:
   Москва: 2024-01-15 14:30:45.123456+03:00
   Нью-Йорк: 2024-01-15 06:30:45.123456-05:00
   Токио: 2024-01-15 20:30:45.123456+09:00
   Смещение MSK: +03:00

2. КОНВЕРТАЦИЯ:
   Москва: 2024-01-15 14:30:45.123456+03:00
   Нью-Йорк: 2024-01-15 06:30:45.123456-05:00

3. ВРЕМЕННЫЕ ИНТЕРВАЛЫ:
   Между 2024-01-01 и 2024-12-31:
   365 дней

4. РАБОЧИЕ ДНИ:
   Рабочих дней в 2024 году: 262
   Через 5 рабочих дней: 2024-01-22

5. ФОРМАТИРОВАНИЕ:
   Человеческий формат: 15 января 2024 года, 14:30
   Краткий формат: 15.01.2024
   ISO: 2024-01-15T14:30:45.123456

6. ПАРСИНГ:
   '15.01.2024' → 2024-01-15 00:00:00
   '2024-01-15 14:30:00' → 2024-01-15 14:30:00

7. ГРАНИЦЫ ПЕРИОДОВ:
   Исходная дата: 2024-06-15 14:30:00
   Начало дня: 2024-06-15 00:00:00
   Начало недели: 2024-06-10 00:00:00
   Начало месяца: 2024-06-01 00:00:00
   Начало года: 2024-01-01 00:00:00

8. ВАЛИДАЦИЯ:
   29.02.2024 существует? True
   29.02.2023 существует? False
   2024 високосный? True
   Дней в феврале 2024: 29

9. ТАЙМЕР:
   Прошло: 0:00:00.501234

============================================================
Демонстрация завершена
============================================================
25 вариантов практической работы (ПЗ 2.6)
Общая структура каждого варианта:
Студент разрабатывает модуль для работы с датами и временем согласно техническому заданию.

Уровни сложности:

Варианты 1-8: базовый (основные операции с датами)

Варианты 9-17: средний (часовые пояса, интервалы)

Варианты 18-25: сложный (полноценные утилиты, производительность)

Варианты 1-8 (Базовый уровень)
Вариант 1. Калькулятор возраста
ТЗ: Разработайте модуль age_calculator.py с функциями:

calculate_age(birth_date) — возраст в годах

days_until_birthday(birth_date) — дней до дня рождения

zodiac_sign(birth_date) — знак зодиака

Вариант 2. Форматтер дат
ТЗ: Разработайте модуль date_formatter.py с функциями:

to_ru_short(date) — DD.MM.YYYY

to_ru_long(date) — DD Month YYYY

to_iso(date) — YYYY-MM-DD

to_custom(date, format) — произвольный формат

Вариант 3. Разница между датами
ТЗ: Разработайте модуль date_diff.py с функциями:

days_between(d1, d2) — количество дней

weeks_between(d1, d2) — количество недель

months_between(d1, d2) — количество месяцев (приближённо)

years_between(d1, d2) — количество лет

Вариант 4. Валидатор дат
ТЗ: Разработайте модуль date_validator.py с функциями:

is_valid_date(year, month, day) — существует ли дата

is_leap_year(year) — високосный ли год

is_weekend(date) — выходной ли день

is_workday(date) — рабочий ли день

Вариант 5. Генератор диапазонов
ТЗ: Разработайте модуль date_range.py с функциями:

date_range(start, end) — генератор дат

month_dates(year, month) — все дни месяца

week_dates(date) — все дни недели

workday_range(start, end) — только рабочие дни

Вариант 6. Парсер дат
ТЗ: Разработайте модуль date_parser.py с функциями:

parse_date(string) — парсинг из DD.MM.YYYY, YYYY-MM-DD

parse_datetime(string) — парсинг с временем

parse_relative(string) — "today", "tomorrow", "yesterday"

Вариант 7. Календарь событий
ТЗ: Разработайте модуль event_calendar.py с классом EventCalendar:

add_event(date, name) — добавить событие

get_events(date) — получить события на дату

get_upcoming(days) — предстоящие события

save(filename) / load(filename) — сохранение/загрузка

Вариант 8. Таймер обратного отсчёта
ТЗ: Разработайте модуль countdown.py с функцией:

countdown(target_date) — обратный отсчёт до даты

Формат вывода: "5 дней 3 часа 15 минут до события"

Поддержка разных единиц измерения

Варианты 9-17 (Средний уровень)
Вариант 9. Конвертер часовых поясов
ТЗ: Разработайте модуль timezone_converter.py с функциями:

convert(dt, from_tz, to_tz) — конвертация между поясами

list_timezones() — список доступных поясов

get_offset(tz) — смещение UTC

now_in(tz) — текущее время в поясе

Вариант 10. Рабочий календарь
ТЗ: Разработайте модуль work_calendar.py с классом WorkCalendar:

Учёт праздничных дней (передаются списком)

business_days_between(start, end) — рабочие дни

add_business_days(date, days) — прибавить рабочие дни

is_holiday(date) — проверка праздника

Вариант 11. Тайм-менеджмент
ТЗ: Разработайте модуль time_manager.py с классом TimeManager:

time_spent(start, end) — потраченное время

total_time(tasks) — суммарное время по задачам

format_duration(seconds) — форматирование

remaining(deadline) — оставшееся время до дедлайна

Вариант 12. Планировщик напоминаний
ТЗ: Разработайте модуль reminder.py с классом Reminder:

set_reminder(datetime, message) — установить напоминание

get_due() — список просроченных

get_upcoming(hours) — предстоящие

check() — проверка и вывод (для демо)

Вариант 13. Анализ временных рядов
ТЗ: Разработайте модуль time_series.py с функциями:

group_by_hour(dates) — группировка по часам

group_by_day(dates) — группировка по дням

group_by_month(dates) — группировка по месяцам

frequency_analysis(dates) — анализ частоты

Вариант 14. Расчёт времени доставки
ТЗ: Разработайте модуль delivery_time.py с функциями:

delivery_date(order_date, days, holidays) — дата доставки (только рабочие дни)

shipping_deadline(delivery_date, days) — крайний срок отправки

is_delivery_possible(order_date, delivery_date) — возможна ли доставка

Вариант 15. Логгер с временными метками
ТЗ: Разработайте модуль timed_logger.py с классом TimedLogger:

log(message) — запись с меткой UTC

get_logs(date) — логи за дату

get_logs_between(start, end) — логи за период

Ротация логов по дням

Вариант 16. Сериализация дат
ТЗ: Разработайте модуль date_serializer.py с функциями:

to_json(dt) — datetime в JSON-строку

from_json(json_str) — из JSON в datetime

to_timestamp(dt) — в Unix timestamp

from_timestamp(ts) — из Unix timestamp

Вариант 17. Возрастная категория
ТЗ: Разработайте модуль age_category.py с функциями:

age_category(birth_date) — категория (ребёнок, подросток, взрослый, пожилой)

generation(birth_date) — поколение (Z, Millennial, X, Boomer)

retirement_date(birth_date) — дата выхода на пенсию (65 лет)

Варианты 18-25 (Сложный уровень)
Вариант 18. Международный календарь
ТЗ: Разработайте модуль world_calendar.py с поддержкой:

Разных систем исчисления (Григорианский, Юлианский)

Преобразование между календарями

Определение праздников по стране

Использование pytz и dateutil

Вариант 19. Тайм-трекер
ТЗ: Разработайте полноценный модуль time_tracker.py:

Запуск/остановка задач

Пауза/возобновление

Отчёты по дням/неделям/месяцам

Экспорт в CSV/JSON

Сохранение состояния между запусками

Вариант 20. Астрологический калькулятор
ТЗ: Разработайте модуль astro_calculator.py:

Солнечные и лунные фазы

Восход/закат для координат

Долгота дня

Использование библиотеки ephem или astral

Вариант 21. Система бронирования переговорных
ТЗ: Разработайте модуль booking_system.py с классом BookingSystem:

Бронирование на времеБронирование на временной слот

Проверка пересечений

Освобождение просроченных броней

Учёт часовых поясов участников

Вариант 22. Расчёт сложных процентов по дням
ТЗ: Разработайте модуль interest_calculator.py:

Расчёт процентов с учётом точного количества дней

Поддержка 365/360 дней (банковский метод)

Капитализация процентов

График платежей

Вариант 23. Парсер логов с временными метками
ТЗ: Разработайте модуль log_parser.py:

Парсинг логов в разных форматах времени

Поиск событий во временном окне

Вычисление времени между событиями

Агрегация по временным интервалам

Обработка тайм-аутов

Вариант 24. Симулятор временной зоны
ТЗ: Разработайте модуль timezone_simulator.py:

Эмуляция работы в разных часовых поясах

Планирование встреч с учётом поясов участников

Поиск оптимального времени для всех

Визуализация временной сетки

Вариант 25. Кроссплатформенные часы реального времени
ТЗ: Разработайте модуль rt_clock.py:

Синхронизация с NTP-сервером

Компенсация дрейфа времени

Планирование событий с точностью до миллисекунды

Генерация временных меток для распределённых систем

Карточка студента (шаблон)
text
ПЗ 2.6. РАЗРАБОТКА МОДУЛЯ ДЛЯ РАБОТЫ С ДАТАМИ И ЧАСОВЫМИ ПОЯСАМИ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Тема модуля: _________________________________

=== ТРЕБОВАНИЯ ===

Обязательный функционал:
1. _______________________________
2. _______________________________
3. _______________________________

Дополнительный функционал:
1. _______________________________

=== ТЕХНИЧЕСКИЕ ТРЕБОВАНИЯ ===

□ Использовать datetime и timedelta
□ Использовать pytz для часовых поясов
□ Добавить аннотации типов
□ Написать docstring для всех функций
□ Добавить примеры использования (doctest)

=== ОТЧЁТ ===

Исходный код модуля: _______________________________
Демонстрация работы: _______________________________
Тесты (5+ примеров): _______________________________

Оценка pylint: ___/10
Сложность radon cc: ___

Дата выполнения: _____________
Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Модуль не работает или содержит критические ошибки
3 (удовлетворительно)	Реализован базовый функционал, есть ошибки с часовыми поясами
4 (хорошо)	Реализован весь обязательный функционал, код соответствует PEP 8
5 (отлично)	Реализован дополнительный функционал, высокая оценка pylint, есть тесты
Шпаргалка по работе с датами
python
# === ОСНОВНЫЕ ИМПОРТЫ ===
from datetime import datetime, date, time, timedelta, timezone
import pytz
from zoneinfo import ZoneInfo  # Python 3.9+

# === СОЗДАНИЕ ===
now = datetime.now()
now_utc = datetime.now(timezone.utc)
now_moscow = datetime.now(pytz.timezone('Europe/Moscow'))

# === ФОРМАТИРОВАНИЕ ===
dt.strftime("%d.%m.%Y %H:%M:%S")
datetime.strptime("15.01.2024", "%d.%m.%Y")

# === АРИФМЕТИКА ===
tomorrow = datetime.now() + timedelta(days=1)
delta = date2 - date1
delta.days
delta.total_seconds()

# === ЧАСОВЫЕ ПОЯСА ===
tz = pytz.timezone('America/New_York')
aware_dt = tz.localize(naive_dt)
converted = aware_dt.astimezone(pytz.UTC)

# === ПОЛЕЗНЫЕ ФУНКЦИИ ===
calendar.monthrange(2024, 2)  # (первый день недели, кол-во дней)
calendar.isleap(2024)  # високосный?нной
