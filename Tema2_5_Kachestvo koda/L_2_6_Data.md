# Тема 2.6. Работа с датой и временем в Python

## Лекция 2.6. datetime, timezone, timedelta

**Цель лекции:**  
Научиться профессионально работать с датами и временем в Python: создавать, сравнивать, преобразовывать, выполнять арифметические операции, учитывать часовые пояса и форматировать для вывода.

> Главная мысль: **Дата и время — одни из самых сложных типов данных. 90% ошибок в продакшене связаны с неправильной работой со временем. Научитесь делать это правильно.**

---

## Часть 1. Модуль datetime: основные типы

### 1.1. Четыре основных класса

| Класс | Описание | Пример |
|-------|----------|--------|
| `date` | Дата (год, месяц, день) | `2024-01-15` |
| `time` | Время (час, минута, секунда, микросекунда) | `14:30:45.123456` |
| `datetime` | Дата + время | `2024-01-15 14:30:45.123456` |
| `timedelta` | Разница между датами/временами | `5 days, 2:30:00` |

### 1.2. Импорт и создание объектов

```python
from datetime import date, time, datetime, timedelta

# === СОЗДАНИЕ ДАТ ===
today = date.today()                    # текущая дата
birthday = date(1995, 5, 15)            # конкретная дата
year, month, day = birthday.year, birthday.month, birthday.day

# === СОЗДАНИЕ ВРЕМЕНИ ===
current_time = time(14, 30, 45)         # 14:30:45
time_with_micro = time(14, 30, 45, 123456)

# === СОЗДАНИЕ DATETIME ===
now = datetime.now()                    # текущие дата и время
specific = datetime(2024, 1, 15, 14, 30, 45)
utc_now = datetime.utcnow()             # UTC время (без часового пояса)

# === ИЗ СТРОКИ ===
from_string = datetime.strptime("2024-01-15 14:30:45", "%Y-%m-%d %H:%M:%S")

# === В СТРОКУ ===
as_string = now.strftime("%d.%m.%Y %H:%M")
1.3. Атрибуты объектов
python
dt = datetime.now()

print(f"Год: {dt.year}")           # 2024
print(f"Месяц: {dt.month}")         # 1-12
print(f"День: {dt.day}")            # 1-31
print(f"Час: {dt.hour}")            # 0-23
print(f"Минута: {dt.minute}")       # 0-59
print(f"Секунда: {dt.second}")      # 0-59
print(f"Микросекунда: {dt.microsecond}")  # 0-999999

# Дополнительно для date
print(f"День недели: {dt.weekday()}")     # 0=пн, 6=вс
print(f"ISO день недели: {dt.isoweekday()}")  # 1=пн, 7=вс
print(f"День в году: {dt.timetuple().tm_yday}")
Часть 2. Форматирование даты и времени (strftime / strptime)
2.1. Основные коды форматирования
Код	Описание	Пример
%Y	Год (4 цифры)	2024
%y	Год (2 цифры)	24
%m	Месяц (01-12)	01
%d	День (01-31)	15
%H	Час (00-23)	14
%I	Час (01-12)	02
%M	Минута (00-59)	30
%S	Секунда (00-59)	45
%f	Микросекунды (6 цифр)	123456
%p	AM/PM	PM
%a	Сокращённый день недели	Mon
%A	Полный день недели	Monday
%b	Сокращённый месяц	Jan
%B	Полный месяц	January
%w	День недели (0-6, 0=вс)	1
%j	День в году (001-366)	015
%W	Номер недели в году	02
%z	Смещение часового пояса	+0300
%Z	Название часового пояса	UTC
2.2. Преобразование datetime → строка (strftime)
python
from datetime import datetime

now = datetime.now()

# Русскоязычный формат
print(now.strftime("%d.%m.%Y %H:%M:%S"))        # 15.01.2024 14:30:45

# ISO формат
print(now.strftime("%Y-%m-%dT%H:%M:%S"))        # 2024-01-15T14:30:45

# Для логов
print(now.strftime("[%Y-%m-%d %H:%M:%S]"))      # [2024-01-15 14:30:45]

# Для имён файлов
print(now.strftime("%Y%m%d_%H%M%S"))            # 20240115_143045

# "Человеческий" формат
print(now.strftime("%A, %d %B %Y года"))        # Monday, 15 January 2024 года

# Время с AM/PM
print(now.strftime("%I:%M %p"))                 # 02:30 PM
2.3. Преобразование строка → datetime (strptime)
python
from datetime import datetime

# Стандартный формат
date_str = "2024-01-15 14:30:45"
dt = datetime.strptime(date_str, "%Y-%m-%d %H:%M:%S")

# Русский формат
date_str = "15.01.2024 14:30:45"
dt = datetime.strptime(date_str, "%d.%m.%Y %H:%M:%S")

# С названиями месяцев
date_str = "15 January 2024"
dt = datetime.strptime(date_str, "%d %B %Y")

# С AM/PM
date_str = "02:30:45 PM"
dt = datetime.strptime(date_str, "%I:%M:%S %p")

# Обработка ошибок
try:
    dt = datetime.strptime("31.02.2024", "%d.%m.%Y")
except ValueError as e:
    print(f"Ошибка: {e}")  # day is out of range for month
2.4. Полезные ISO форматы
python
from datetime import datetime

now = datetime.now()

# ISO 8601 формат (стандарт)
iso_string = now.isoformat()                    # 2024-01-15T14:30:45.123456

# Только дата в ISO
date_iso = now.date().isoformat()               # 2024-01-15

# Только время в ISO
time_iso = now.time().isoformat()               # 14:30:45.123456

# Из ISO строки
dt = datetime.fromisoformat("2024-01-15T14:30:45")
Часть 3. Арифметика с датами (timedelta)
3.1. Создание timedelta
python
from datetime import datetime, timedelta

# Разные способы создания
delta1 = timedelta(days=5)                      # 5 дней
delta2 = timedelta(hours=3)                     # 3 часа
delta3 = timedelta(minutes=30)                  # 30 минут
delta4 = timedelta(weeks=2)                     # 2 недели
delta5 = timedelta(days=1, hours=2, minutes=30)  # 1 день 2 часа 30 минут

# Комбинированный
complex_delta = timedelta(
    days=10,
    seconds=3600,      # 1 час
    microseconds=1000,
    milliseconds=500,
    minutes=30,
    hours=2,
    weeks=1
)
3.2. Операции с timedelta
python
from datetime import datetime, timedelta

now = datetime.now()

# ===== ПРИБАВЛЕНИЕ И ВЫЧИТАНИЕ =====
tomorrow = now + timedelta(days=1)
yesterday = now - timedelta(days=1)
next_week = now + timedelta(weeks=1)
last_month = now - timedelta(days=30)
in_3_hours = now + timedelta(hours=3)

# ===== СРАВНЕНИЕ =====
is_future = tomorrow > now                      # True
is_past = yesterday < now                       # True

# ===== РАЗНИЦА МЕЖДУ ДАТАМИ =====
start_date = datetime(2024, 1, 1)
end_date = datetime(2024, 12, 31)
diff = end_date - start_date                    # timedelta

print(f"Дней: {diff.days}")                     # 365
print(f"Секунд: {diff.seconds}")                # 0
print(f"Всего секунд: {diff.total_seconds()}")  # 31536000.0

# ===== ОКРУГЛЕНИЕ =====
from math import floor, ceil

diff_days = diff.total_seconds() / (24 * 3600)
print(f"Точное количество дней: {diff_days}")   # 365.0
3.3. Практические примеры с timedelta
python
from datetime import datetime, timedelta

def days_until_birthday(birthday: date) -> int:
    """Дней до следующего дня рождения."""
    today = date.today()
    next_birthday = date(today.year, birthday.month, birthday.day)
    
    if next_birthday < today:
        next_birthday = date(today.year + 1, birthday.month, birthday.day)
    
    return (next_birthday - today).days

def is_expired(expiry_date: datetime, grace_days: int = 0) -> bool:
    """Проверяет, истёк ли срок действия."""
    return datetime.now() > expiry_date + timedelta(days=grace_days)

def format_duration(delta: timedelta) -> str:
    """Форматирует timedelta в человеко-читаемый вид."""
    days = delta.days
    hours = delta.seconds // 3600
    minutes = (delta.seconds % 3600) // 60
    seconds = delta.seconds % 60
    
    parts = []
    if days:
        parts.append(f"{days} д")
    if hours:
        parts.append(f"{hours} ч")
    if minutes:
        parts.append(f"{minutes} мин")
    if seconds or not parts:
        parts.append(f"{seconds} сек")
    
    return " ".join(parts)

# Пример использования
delta = timedelta(days=5, hours=3, minutes=15)
print(format_duration(delta))  # "5 д 3 ч 15 мин"
Часть 4. Работа с часовыми поясами (timezone)
4.1. Наивные vs осведомлённые datetime
python
from datetime import datetime, timezone, timedelta
import pytz  # pip install pytz

# НАИВНЫЙ (naive) — без часового пояса
naive_now = datetime.now()              # нет информации о часовом поясе

# ОСВЕДОМЛЁННЫЙ (aware) — с часовым поясом
utc_now = datetime.now(timezone.utc)    # UTC время с часовым поясом

# Разница критична для международных приложений
print(naive_now.tzinfo)                 # None
print(utc_now.tzinfo)                   # UTC
4.2. Библиотека pytz (рекомендована)
python
from datetime import datetime
import pytz

# ===== ПОЛУЧЕНИЕ ЧАСОВЫХ ПОЯСОВ =====
# Все часовые пояса
all_timezones = pytz.all_timezones       # список из ~600 поясов

# Конкретные пояса
utc = pytz.UTC
moscow = pytz.timezone('Europe/Moscow')
new_york = pytz.timezone('America/New_York')
tokyo = pytz.timezone('Asia/Tokyo')
london = pytz.timezone('Europe/London')

# ===== СОЗДАНИЕ ОСВЕДОМЛЁННЫХ DATETIME =====
# Способ 1: локальное время с указанием пояса
dt_moscow = datetime(2024, 1, 15, 14, 30, 0, tzinfo=moscow)

# Способ 2: из UTC
dt_utc = datetime(2024, 1, 15, 11, 30, 0, tzinfo=utc)

# Способ 3: локализация наивного datetime (ПРАВИЛЬНО)
naive = datetime(2024, 1, 15, 14, 30, 0)
dt_moscow = moscow.localize(naive)

# ===== КОНВЕРТАЦИЯ МЕЖДУ ПОЯСАМИ =====
# Создаём время в Москве
moscow_tz = pytz.timezone('Europe/Moscow')
moscow_time = moscow_tz.localize(datetime(2024, 1, 15, 14, 30, 0))

# Конвертируем в Нью-Йорк
ny_tz = pytz.timezone('America/New_York')
ny_time = moscow_time.astimezone(ny_tz)

print(f"Москва: {moscow_time}")        # 2024-01-15 14:30:00+03:00
print(f"Нью-Йорк: {ny_time}")          # 2024-01-15 06:30:00-05:00
4.3. zoneinfo (Python 3.9+, альтернатива pytz)
python
from datetime import datetime, timezone
from zoneinfo import ZoneInfo  # Python 3.9+

# zoneinfo встроен, не требует установки
moscow = ZoneInfo("Europe/Moscow")
ny = ZoneInfo("America/New_York")

# Создание осведомлённого datetime
dt_moscow = datetime(2024, 1, 15, 14, 30, 0, tzinfo=moscow)

# Конвертация
dt_ny = dt_moscow.astimezone(ny)

# Текущее время в разных поясах
now_moscow = datetime.now(moscow)
now_ny = datetime.now(ny)
now_utc = datetime.now(timezone.utc)
4.4. Работа с временными метками (timestamp)
python
from datetime import datetime, timezone
import time

# ===== UNIX TIMESTAMP =====
# Текущий timestamp
timestamp = time.time()                     # 1705318245.123456

# Из datetime в timestamp
dt = datetime.now()
timestamp = dt.timestamp()

# Из timestamp в datetime
dt_from_ts = datetime.fromtimestamp(timestamp)

# UTC timestamp
utc_dt = datetime.now(timezone.utc)
utc_timestamp = utc_dt.timestamp()

# ===== ПРИМЕР: ХРАНЕНИЕ В БАЗЕ ДАННЫХ =====
# Рекомендация: всегда хранить в UTC
def store_event(event_name: str):
    """Сохраняет событие с меткой времени UTC."""
    utc_time = datetime.now(timezone.utc)
    timestamp = int(utc_time.timestamp())   # целое число секунд
    
    # Сохраняем timestamp в БД
    # INSERT INTO events (name, created_at) VALUES (?, ?)
    print(f"Событие '{event_name}' в {timestamp}")

def get_events_since(days: int):
    """Получает события за последние N дней."""
    since_timestamp = int(time.time()) - days * 86400
    # SELECT * FROM events WHERE created_at > ?
    return since_timestamp
4.5. Типичные проблемы с часовыми поясами
python
from datetime import datetime
import pytz

# ПРОБЛЕМА 1: Наивное сравнение с осведомлённым
naive = datetime.now()
aware = datetime.now(pytz.UTC)
# naive > aware  # TypeError! Нельзя сравнивать

# Решение: привести к общему типу
aware_naive = naive.replace(tzinfo=pytz.UTC)

# ПРОБЛЕМА 2: Переход на летнее время
moscow = pytz.timezone('Europe/Moscow')
# В России больше нет перевода, но в других странах есть

# ПРОБЛЕМА 3: Несуществующее время (при переводе часов)
# 2:30 ночи в день перевода может не существовать
try:
    dt = moscow.localize(datetime(2024, 3, 31, 2, 30, 0))
except pytz.exceptions.NonExistentTimeError:
    print("Этого времени не существует!")

# Решение: использовать параметры is_dst
dt = moscow.localize(datetime(2024, 3, 31, 2, 30, 0), is_dst=False)
Часть 5. Практические примеры и утилиты
5.1. Генерация диапазонов дат
python
from datetime import datetime, timedelta, date

def date_range(start_date: date, end_date: date):
    """Генерирует все даты между start и end включительно."""
    current = start_date
    while current <= end_date:
        yield current
        current += timedelta(days=1)

# Пример: все дни месяца
def days_in_month(year: int, month: int):
    """Возвращает список всех дней месяца."""
    first_day = date(year, month, 1)
    if month == 12:
        last_day = date(year + 1, 1, 1) - timedelta(days=1)
    else:
        last_day = date(year, month + 1, 1) - timedelta(days=1)
    
    return list(date_range(first_day, last_day))

# Использование
for day in date_range(date(2024, 1, 1), date(2024, 1, 10)):
    print(day.strftime("%d.%m.%Y"))
5.2. Работа с рабочими днями
python
from datetime import date, timedelta

def is_weekend(day: date) -> bool:
    """Проверяет, является ли день выходным."""
    return day.weekday() >= 5  # 5=суббота, 6=воскресенье

def add_workdays(start_date: date, days: int) -> date:
    """Добавляет N рабочих дней к дате."""
    current = start_date
    added = 0
    
    while added < days:
        current += timedelta(days=1)
        if not is_weekend(current):
            added += 1
    
    return current

def get_workdays_between(start: date, end: date) -> int:
    """Считает количество рабочих дней между датами."""
    if start > end:
        start, end = end, start
    
    current = start
    workdays = 0
    
    while current <= end:
        if not is_weekend(current):
            workdays += 1
        current += timedelta(days=1)
    
    return workdays

# Использование
today = date.today()
next_friday = add_workdays(today, 5)
print(f"Через 5 рабочих дней: {next_friday}")
5.3. Класс для работы со временем
python
from datetime import datetime, timedelta, timezone
from typing import Optional, Union
import json

class DateTimeUtils:
    """Утилиты для работы с датой и временем."""
    
    @staticmethod
    def now_utc() -> datetime:
        """Возвращает текущее UTC время."""
        return datetime.now(timezone.utc)
    
    @staticmethod
    def to_iso(dt: datetime) -> str:
        """Преобразует datetime в ISO строку."""
        return dt.isoformat()
    
    @staticmethod
    def from_iso(iso_string: str) -> datetime:
        """Создаёт datetime из ISO строки."""
        return datetime.fromisoformat(iso_string)
    
    @staticmethod
    def human_readable(dt: datetime) -> str:
        """Возвращает человеко-читаемый формат."""
        return dt.strftime("%d.%m.%Y в %H:%M")
    
    @staticmethod
    def age(birth_date: date) -> int:
        """Вычисляет возраст в годах."""
        today = date.today()
        age = today.year - birth_date.year
        if (today.month, today.day) < (birth_date.month, birth_date.day):
            age -= 1
        return age
    
    @staticmethod
    def start_of_day(dt: datetime) -> datetime:
        """Возвращает начало дня (00:00:00)."""
        return datetime(dt.year, dt.month, dt.day)
    
    @staticmethod
    def end_of_day(dt: datetime) -> datetime:
        """Возвращает конец дня (23:59:59.999999)."""
        return datetime(dt.year, dt.month, dt.day, 23, 59, 59, 999999)
    
    @staticmethod
    def days_between(d1: date, d2: date) -> int:
        """Количество дней между датами."""
        return abs((d2 - d1).days)
    
    @staticmethod
    def to_json(dt: datetime) -> str:
        """Сериализует datetime в JSON строку."""
        return dt.isoformat()
    
    @staticmethod
    def from_json(json_str: str) -> datetime:
        """Десериализует datetime из JSON строки."""
        return datetime.fromisoformat(json_str)

# Пример использования
utils = DateTimeUtils()
print(f"UTC сейчас: {utils.now_utc()}")
print(f"Человеческий формат: {utils.human_readable(datetime.now())}")
print(f"Возраст: {utils.age(date(1995, 5, 15))} лет")
Часть 6. Работа с датами в реальных задачах
6.1. Логирование с временными метками
python
import logging
from datetime import datetime, timezone
from pathlib import Path

class TimedLogger:
    """Логгер с временными метками в UTC."""
    
    def __init__(self, log_dir: Path):
        self.log_dir = log_dir
        self.log_dir.mkdir(exist_ok=True)
    
    def _get_log_filename(self) -> Path:
        """Имя файла лога = текущая дата."""
        today = datetime.now(timezone.utc).strftime("%Y-%m-%d")
        return self.log_dir / f"{today}.log"
    
    def log(self, level: str, message: str):
        """Записывает сообщение в лог."""
        timestamp = datetime.now(timezone.utc).isoformat()
        log_line = f"[{timestamp}] {level}: {message}\n"
        
        with open(self._get_log_filename(), 'a', encoding='utf-8') as f:
            f.write(log_line)
        
        print(log_line.strip())
    
    def get_logs_for_date(self, date: date) -> list:
        """Читает логи за конкретную дату."""
        log_file = self.log_dir / f"{date.isoformat()}.log"
        if not log_file.exists():
            return []
        
        return log_file.read_text(encoding='utf-8').splitlines()

# Использование
logger = TimedLogger(Path("./logs"))
logger.log("INFO", "Приложение запущено")
logger.log("ERROR", "Ошибка подключения к БД")
6.2. Планировщик задач по расписанию
python
from datetime import datetime, time, timedelta
from typing import Callable, List
import time as time_module

class ScheduledTask:
    """Задача, выполняемая по расписанию."""
    
    def __init__(self, name: str, func: Callable, schedule_time: time):
        self.name = name
        self.func = func
        self.schedule_time = schedule_time
        self.last_run: datetime = None
    
    def should_run(self, now: datetime) -> bool:
        """Проверяет, нужно ли выполнить задачу."""
        # Не запускать, если уже запускали сегодня
        if self.last_run and self.last_run.date() == now.date():
            return False
        
        # Проверяем время
        return now.time() >= self.schedule_time
    
    def run(self, now: datetime):
        """Выполняет задачу."""
        self.func()
        self.last_run = now
        print(f"[{now}] Задача '{self.name}' выполнена")

class Scheduler:
    """Планировщик задач."""
    
    def __init__(self):
        self.tasks: List[ScheduledTask] = []
    
    def add_task(self, task: ScheduledTask):
        self.tasks.append(task)
    
    def run_forever(self, check_interval: int = 60):
        """Запускает бесконечный цикл проверки задач."""
        print("Планировщик запущен")
        
        while True:
            now = datetime.now()
            
            for task in self.tasks:
                if task.should_run(now):
                    task.run(now)
            
            time_module.sleep(check_interval)

# Пример
def backup_database():
    print("  → Создание бэкапа базы данных...")

def send_daily_report():
    print("  → Отправка дневного отчёта...")

# Настройка
scheduler = Scheduler()
scheduler.add_task(ScheduledTask("Бэкап", backup_database, time(2, 0)))   # в 02:00
scheduler.add_task(ScheduledTask("Отчёт", send_daily_report, time(9, 30))) # в 09:30

# Запуск (закомментировано для демонстрации)
# scheduler.run_forever()
6.3. Валидация дат и времени
python
from datetime import datetime, date
from typing import Optional

def validate_date(year: int, month: int, day: int) -> bool:
    """Проверяет, существует ли дата."""
    try:
        date(year, month, day)
        return True
    except ValueError:
        return False

def parse_flexible_date(date_string: str) -> Optional[datetime]:
    """Парсит дату в разных форматах."""
    formats = [
        "%Y-%m-%d",
        "%d.%m.%Y",
        "%d/%m/%Y",
        "%Y%m%d",
        "%d-%m-%Y",
        "%Y-%m-%d %H:%M:%S",
        "%d.%m.%Y %H:%M",
    ]
    
    for fmt in formats:
        try:
            return datetime.strptime(date_string, fmt)
        except ValueError:
            continue
    
    return None

def is_valid_date_range(start: date, end: date) -> bool:
    """Проверяет, что диапазон дат корректен."""
    if start > end:
        return False
    if end < start:
        return False
    if (end - start).days > 365 * 10:  # максимум 10 лет
        return False
    return True

# Использование
print(validate_date(2024, 2, 29))  # True (високосный)
print(validate_date(2023, 2, 29))  # False

parsed = parse_flexible_date("15.01.2024")
print(f"Распознано: {parsed}")

range_valid = is_valid_date_range(date(2024, 1, 1), date(2024, 12, 31))
print(f"Диапазон корректен: {range_valid}")
Шпаргалка по datetime
python
# === ИМПОРТЫ ===
from datetime import datetime, date, time, timedelta, timezone
import pytz  # для часовых поясов

# === СОЗДАНИЕ ===
datetime.now()                           # текущие дата+время
datetime.today()                         # текущие (без микросекунд)
datetime.utcnow()                        # UTC (наивное!)
date.today()                             # сегодняшняя дата
datetime(2024, 1, 15, 14, 30)           # конкретные

# === ФОРМАТИРОВАНИЕ ===
dt.strftime("%d.%m.%Y %H:%M")           # datetime → строка
datetime.strptime("15.01.2024", "%d.%m.%Y")  # строка → datetime
dt.isoformat()                           # ISO 8601
datetime.fromisoformat("2024-01-15T14:30")

# === АРИФМЕТИКА ===
dt + timedelta(days=5)                   # будущая дата
dt - timedelta(hours=3)                  # прошлая дата
dt2 - dt1                                # разница (timedelta)
(dt2 - dt1).days                         # количество дней
(dt2 - dt1).total_seconds()              # количество секунд

# === ЧАСОВЫЕ ПОЯСА ===
datetime.now(timezone.utc)               # текущее UTC (aware)
pytz.timezone('Europe/Moscow')           # объект часового пояса
tz.localize(naive_dt)                    # наивный → осведомлённый
dt_aware.astimezone(other_tz)            # конвертация между поясами

# === СРАВНЕНИЕ ===
dt1 < dt2                                # сравнение дат
(dt1 - dt2).days > 0                     # разница в днях

# === ПОЛЕЗНЫЕ АТРИБУТЫ ===
dt.year, dt.month, dt.day                # компоненты даты
dt.hour, dt.minute, dt.second            # компоненты времени
dt.weekday()                             # день недели (0=пн)
dt.isoweekday()                          # день недели (1=пн)
Контрольные вопросы
В чём разница между datetime.now() и datetime.utcnow()? Почему лучше использовать UTC?

Как преобразовать строку "15.01.2024 14:30:45" в объект datetime?

Что такое наивный (naive) и осведомлённый (aware) datetime? В чём проблема их сравнения?

Как добавить 7 дней к текущей дате? Как вычесть 3 часа?

Зачем нужен модуль pytz? Что он добавляет к стандартному datetime?

Что произойдёт при попытке создать datetime для 31 февраля? Как это обработать?

Как получить количество дней между двумя датами? Как получить количество секунд?

Какой формат даты рекомендуется использовать для имён файлов и почему?

Почему при переходе на летнее время могут возникать ошибки NonExistentTimeError?

Как получить текущий timestamp (Unix time) и преобразовать его обратно в datetime?

Практические задания (для закрепления)
Задание 1 (базовое)
Напишите функцию days_until_new_year(), которая возвращает количество дней до следующего Нового года.

Задание 2 (среднее)
Напишите функцию parse_user_date(date_string), которая принимает строку в любом из форматов ("2024-01-15", "15.01.2024", "15/01/2024") и возвращает объект date. При ошибке возвращает None.

Задание 3 (сложное)
Реализуйте класс EventScheduler, который позволяет:

Добавлять события с датой и временем

Получать все события на определённую дату

Получать ближайшее предстоящее событие

Удалять прошедшие события

Сохранять события в JSON-файл

Итог лекции
Вы сегодня:

Освоили основные классы модуля datetime

Научились форматировать и парсить даты

Освоили арифметику с датами через timedelta

Поняли разницу между наивными и осведомлёнными datetime

Научились работать с часовыми поясами (pytz, zoneinfo)

Разобрали практические примеры для реальных задач

Теперь вы профессионально работаете с датами и временем — одной из самых сложных тем в программировании!
