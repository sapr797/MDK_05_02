# ПЗ 2.17. Разработка ETL-скриптов: CSV → JSON и JSON → CSV

**Тема:** Концепция ETL, работа с форматами CSV и JSON

**Цель работы:**  
Научиться разрабатывать ETL-скрипты для преобразования данных между форматами CSV и JSON, выполнять извлечение, трансформацию и загрузку данных, обрабатывать ошибки и валидировать данные.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+

> Главная мысль: **Данные редко бывают в том формате, который вам нужен. ETL — это искусство преобразования.**

---

## Содержание

1. [Теоретическая справка](#1-теоретическая-справка)
2. [Нулевой вариант (эталонный)](#2-нулевой-вариант-эталонный)
3. [25 вариантов практической работы](#3-25-вариантов-практической-работы)
4. [Критерии оценки](#4-критерии-оценки)
5. [Шпаргалка](#5-шпаргалка)

---

## 1. Теоретическая справка

### 1.1. Сравнение форматов CSV и JSON

| Характеристика | CSV | JSON |
|----------------|-----|------|
| **Структура** | Плоская (табличная) | Иерархическая (дерево) |
| **Поддержка вложенности** | Нет | Да |
| **Размер файла** | Меньше | Больше (из-за скобок и кавычек) |
| **Читаемость для человека** | Хорошая | Хорошая (с форматированием) |
| **Поддержка типов данных** | Все строки | Числа, строки, булевы, null |
| **Скорость парсинга** | Быстрее | Медленнее |

### 1.2. Типовые сценарии преобразования

| Направление | Когда используется |
|-------------|---------------------|
| **CSV → JSON** | Экспорт из Excel, подготовка для API, веб-приложения |
| **JSON → CSV** | Импорт в Excel, аналитика, отчёты, базы данных |

### 1.3. Структура ETL-скрипта
┌─────────────────────────────────────────────────────────────────┐
│ ETL СКРИПТ │
├─────────────────────────────────────────────────────────────────┤
│ EXTRACT (Извлечение) │
│ ├── Чтение исходного файла │
│ ├── Проверка существования │
│ └── Обработка ошибок чтения │
├─────────────────────────────────────────────────────────────────┤
│ TRANSFORM (Преобразование) │
│ ├── Валидация данных │
│ ├── Очистка (пробелы, null, дубликаты) │
│ ├── Нормализация (регистр, форматы) │
│ ├── Обогащение (вычисляемые поля) │
│ └── Фильтрация (удаление некорректных записей) │
├─────────────────────────────────────────────────────────────────┤
│ LOAD (Загрузка) │
│ ├── Создание выходного файла │
│ ├── Форматирование вывода │
│ ├── Сохранение │
│ └── Отчёт о результате │
└─────────────────────────────────────────────────────────────────┘

text

---

## 2. Нулевой вариант (эталонный)

**Назначение:** преподаватель выполняет этот вариант на занятии, показывая разработку ETL-скриптов.

### Техническое задание (нулевой вариант)

> Разработайте два ETL-скрипта:
> 1. `csv_to_json.py` — преобразование CSV файла с данными о сотрудниках в JSON
> 2. `json_to_csv.py` — преобразование JSON файла с данными о заказах в CSV
>
> Скрипты должны включать этапы извлечения, трансформации и загрузки данных с обработкой ошибок.

### Исходные данные

**Файл `employees.csv` (исходный для CSV → JSON):**

```csv
id,name,email,department,salary,hire_date
1,Иван Иванов,ivan@example.com,IT,75000,2020-01-15
2,Мария Петрова,maria@example.com,HR,65000,2021-03-20
3,Петр Сидоров,petr@example.com,IT,80000,2019-11-10
4,Елена Смирнова,,Marketing,70000,2022-02-01
5,Алексей Козлов,alex@example.com,IT,90000,2018-07-25
Файл orders.json (исходный для JSON → CSV):

json
{
    "orders": [
        {
            "order_id": 101,
            "customer": {
                "name": "Иван Петров",
                "email": "ivan@example.com"
            },
            "items": [
                {"product": "Ноутбук", "price": 50000, "quantity": 1},
                {"product": "Мышь", "price": 1500, "quantity": 2}
            ],
            "status": "delivered"
        },
        {
            "order_id": 102,
            "customer": {
                "name": "Мария Сидорова",
                "email": "maria@example.com"
            },
            "items": [
                {"product": "Клавиатура", "price": 3000, "quantity": 1}
            ],
            "status": "pending"
        }
    ]
}
Эталонная реализация
Скрипт 1: CSV → JSON
python
#!/usr/bin/env python3
"""
csv_to_json.py — ETL-скрипт для преобразования CSV в JSON.

Использование:
    python csv_to_json.py --input employees.csv --output employees.json
    python csv_to_json.py --input employees.csv --output employees.json --pretty
"""

import csv
import json
import argparse
import sys
from pathlib import Path
from datetime import datetime
from typing import List, Dict, Any, Optional


# ============================================================
# ФАЗА 1: EXTRACT (ИЗВЛЕЧЕНИЕ)
# ============================================================

def extract_from_csv(filepath: Path) -> Optional[List[Dict[str, Any]]]:
    """
    Извлекает данные из CSV файла.
    
    Args:
        filepath: Путь к CSV файлу
    
    Returns:
        Список словарей с данными или None при ошибке
    
    Raises:
        FileNotFoundError: Если файл не найден
        ValueError: Если файл пустой или имеет неверный формат
    """
    print(f"\n📂 EXTRACT: Чтение файла {filepath}")
    
    if not filepath.exists():
        raise FileNotFoundError(f"Файл {filepath} не найден")
    
    data = []
    
    try:
        with open(filepath, 'r', encoding='utf-8') as f:
            # Проверка на пустой файл
            first_char = f.read(1)
            if not first_char:
                raise ValueError("Файл пуст")
            f.seek(0)
            
            reader = csv.DictReader(f)
            
            # Проверка на наличие заголовков
            if not reader.fieldnames:
                raise ValueError("Файл не содержит заголовков")
            
            for row_num, row in enumerate(reader, start=2):
                data.append(row)
            
            print(f"   ✅ Извлечено {len(data)} записей")
            return data
            
    except csv.Error as e:
        raise ValueError(f"Ошибка парсинга CSV: {e}")


# ============================================================
# ФАЗА 2: TRANSFORM (ПРЕОБРАЗОВАНИЕ)
# ============================================================

def validate_email(email: str) -> bool:
    """Проверяет корректность email."""
    if not email:
        return False
    return '@' in email and '.' in email


def validate_salary(salary: str) -> Optional[int]:
    """Преобразует зарплату в число, возвращает None при ошибке."""
    try:
        return int(salary)
    except (ValueError, TypeError):
        return None


def transform_employee_data(raw_data: List[Dict[str, str]]) -> List[Dict[str, Any]]:
    """
    Преобразует сырые данные о сотрудниках.
    
    Операции:
    - Очистка пробелов
    - Преобразование типов
    - Валидация email
    - Вычисление стажа работы
    - Фильтрация некорректных записей
    """
    print(f"\n🔄 TRANSFORM: Обработка данных")
    
    transformed = []
    errors = []
    
    for idx, record in enumerate(raw_data):
        # Очистка пробелов
        cleaned = {
            k: v.strip() if v else '' for k, v in record.items()
        }
        
        # Преобразование ID в число
        try:
            employee_id = int(cleaned.get('id', 0))
        except ValueError:
            errors.append(f"Строка {idx + 2}: некорректный ID '{cleaned.get('id')}'")
            continue
        
        # Преобразование зарплаты
        salary = validate_salary(cleaned.get('salary', ''))
        if salary is None:
            errors.append(f"Строка {idx + 2}: некорректная зарплата '{cleaned.get('salary')}'")
            continue
        
        # Валидация email (необязательное поле)
        email = cleaned.get('email', '')
        if email and not validate_email(email):
            errors.append(f"Строка {idx + 2}: некорректный email '{email}'")
            continue
        
        # Преобразование даты приёма
        hire_date_str = cleaned.get('hire_date', '')
        hire_date = None
        try:
            if hire_date_str:
                hire_date = datetime.strptime(hire_date_str, '%Y-%m-%d').date()
        except ValueError:
            errors.append(f"Строка {idx + 2}: некорректная дата '{hire_date_str}'")
            continue
        
        # Вычисление стажа
        years_of_service = None
        if hire_date:
            today = datetime.now().date()
            years_of_service = today.year - hire_date.year
            if (today.month, today.day) < (hire_date.month, hire_date.day):
                years_of_service -= 1
        
        # Формирование итоговой записи
        transformed.append({
            'id': employee_id,
            'name': cleaned.get('name', ''),
            'email': email if email else None,
            'department': cleaned.get('department', ''),
            'salary': salary,
            'hire_date': hire_date.isoformat() if hire_date else None,
            'years_of_service': years_of_service,
            'is_active': True
        })
    
    print(f"   ✅ Успешно обработано: {len(transformed)} записей")
    if errors:
        print(f"   ⚠️ Ошибок: {len(errors)}")
        for error in errors[:5]:  # Показываем первые 5
            print(f"      • {error}")
        if len(errors) > 5:
            print(f"      • ... и ещё {len(errors) - 5} ошибок")
    
    return transformed


# ============================================================
# ФАЗА 3: LOAD (ЗАГРУЗКА)
# ============================================================

def load_to_json(data: List[Dict[str, Any]], output_path: Path, pretty: bool = True) -> None:
    """
    Загружает данные в JSON файл.
    
    Args:
        data: Данные для сохранения
        output_path: Путь к выходному файлу
        pretty: Использовать красивое форматирование
    """
    print(f"\n💾 LOAD: Сохранение в {output_path}")
    
    # Создание структуры с метаданными
    output = {
        'metadata': {
            'source': 'CSV transformation',
            'timestamp': datetime.now().isoformat(),
            'total_records': len(data),
            'fields': list(data[0].keys()) if data else []
        },
        'employees': data
    }
    
    # Создание папки, если её нет
    output_path.parent.mkdir(parents=True, exist_ok=True)
    
    # Сохранение
    with open(output_path, 'w', encoding='utf-8') as f:
        if pretty:
            json.dump(output, f, indent=2, ensure_ascii=False)
        else:
            json.dump(output, f, ensure_ascii=False)
    
    print(f"   ✅ Сохранено {len(data)} записей")
    print(f"   📁 Файл: {output_path.absolute()}")
    print(f"   📊 Размер: {output_path.stat().st_size:,} байт")


# ============================================================
# ОСНОВНАЯ ФУНКЦИЯ
# ============================================================

def main():
    parser = argparse.ArgumentParser(
        description='ETL-скрипт: преобразование CSV в JSON',
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Примеры:
  python csv_to_json.py -i employees.csv -o employees.json
  python csv_to_json.py -i employees.csv -o employees.json --minify
        """
    )
    parser.add_argument('-i', '--input', required=True, help='Входной CSV файл')
    parser.add_argument('-o', '--output', required=True, help='Выходной JSON файл')
    parser.add_argument('--minify', action='store_true', help='Минифицированный JSON (без отступов)')
    
    args = parser.parse_args()
    
    input_path = Path(args.input)
    output_path = Path(args.output)
    
    print("=" * 60)
    print("CSV → JSON ETL Pipeline")
    print("=" * 60)
    
    try:
        # EXTRACT
        raw_data = extract_from_csv(input_path)
        
        if not raw_data:
            print("❌ Нет данных для обработки")
            sys.exit(1)
        
        # TRANSFORM
        transformed_data = transform_employee_data(raw_data)
        
        if not transformed_data:
            print("❌ После трансформации не осталось корректных записей")
            sys.exit(1)
        
        # LOAD
        load_to_json(transformed_data, output_path, pretty=not args.minify)
        
        print("\n" + "=" * 60)
        print("✅ ETL ПРОЦЕСС ЗАВЕРШЁН УСПЕШНО!")
        print("=" * 60)
        
    except FileNotFoundError as e:
        print(f"\n❌ Ошибка: {e}")
        sys.exit(1)
    except ValueError as e:
        print(f"\n❌ Ошибка данных: {e}")
        sys.exit(1)
    except Exception as e:
        print(f"\n❌ Непредвиденная ошибка: {e}")
        sys.exit(1)


if __name__ == "__main__":
    main()
Скрипт 2: JSON → CSV
python
#!/usr/bin/env python3
"""
json_to_csv.py — ETL-скрипт для преобразования JSON в CSV.

Использование:
    python json_to_csv.py --input orders.json --output orders.csv
"""

import csv
import json
import argparse
import sys
from pathlib import Path
from datetime import datetime
from typing import List, Dict, Any, Optional


# ============================================================
# ФАЗА 1: EXTRACT (ИЗВЛЕЧЕНИЕ)
# ============================================================

def extract_from_json(filepath: Path) -> Optional[List[Dict[str, Any]]]:
    """
    Извлекает данные из JSON файла.
    
    Args:
        filepath: Путь к JSON файлу
    
    Returns:
        Список словарей с данными или None при ошибке
    """
    print(f"\n📂 EXTRACT: Чтение файла {filepath}")
    
    if not filepath.exists():
        raise FileNotFoundError(f"Файл {filepath} не найден")
    
    try:
        with open(filepath, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        # Определяем структуру данных
        if isinstance(data, dict):
            # Ищем массив данных (возможные ключи: orders, data, items, results)
            for key in ['orders', 'data', 'items', 'results', 'records']:
                if key in data and isinstance(data[key], list):
                    records = data[key]
                    print(f"   ✅ Найден массив '{key}' с {len(records)} записями")
                    return records
            print(f"   ⚠️ Неизвестная структура: {list(data.keys())}")
            return [data]  # Предполагаем, что это одна запись
        
        elif isinstance(data, list):
            print(f"   ✅ Найден массив с {len(data)} записями")
            return data
        
        else:
            raise ValueError(f"Неожиданный тип данных: {type(data)}")
            
    except json.JSONDecodeError as e:
        raise ValueError(f"Ошибка парсинга JSON: {e}")


# ============================================================
# ФАЗА 2: TRANSFORM (ПРЕОБРАЗОВАНИЕ)
# ============================================================

def flatten_json(data: Dict[str, Any], parent_key: str = '', sep: str = '_') -> Dict[str, Any]:
    """
    Преобразует вложенный JSON в плоский словарь.
    
    Example:
        {"customer": {"name": "Иван", "email": "ivan@example.com"}}
        → {"customer_name": "Иван", "customer_email": "ivan@example.com"}
    """
    items = []
    for k, v in data.items():
        new_key = f"{parent_key}{sep}{k}" if parent_key else k
        if isinstance(v, dict):
            items.extend(flatten_json(v, new_key, sep=sep).items())
        elif isinstance(v, list):
            # Для списков: преобразуем в JSON строку
            items.append((new_key, json.dumps(v, ensure_ascii=False)))
        else:
            items.append((new_key, v))
    return dict(items)


def calculate_order_total(order: Dict[str, Any]) -> float:
    """Вычисляет общую сумму заказа."""
    items = order.get('items', [])
    total = 0
    for item in items:
        price = item.get('price', 0)
        quantity = item.get('quantity', 0)
        total += price * quantity
    return total


def transform_order_data(raw_data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
    """
    Преобразует сырые данные о заказах.
    
    Операции:
    - Преобразование вложенных структур в плоские
    - Вычисление суммы заказа
    - Нормализация статуса
    - Очистка данных
    """
    print(f"\n🔄 TRANSFORM: Обработка данных")
    
    transformed = []
    errors = []
    
    for idx, record in enumerate(raw_data):
        try:
            # Преобразование вложенных структур
            flat_record = flatten_json(record)
            
            # Вычисление общей суммы
            total_amount = calculate_order_total(record)
            flat_record['total_amount'] = total_amount
            
            # Нормализация статуса
            status = flat_record.get('status', 'unknown').lower()
            status_map = {
                'delivered': 'Доставлен',
                'pending': 'В обработке',
                'cancelled': 'Отменён',
                'shipped': 'Отправлен'
            }
            flat_record['status_ru'] = status_map.get(status, status)
            
            # Добавление временной метки обработки
            flat_record['processed_at'] = datetime.now().isoformat()
            
            # Очистка null значений
            flat_record = {k: (v if v is not None else '') for k, v in flat_record.items()}
            
            transformed.append(flat_record)
            
        except Exception as e:
            errors.append(f"Запись {idx}: {e}")
    
    print(f"   ✅ Успешно обработано: {len(transformed)} записей")
    if errors:
        print(f"   ⚠️ Ошибок: {len(errors)}")
        for error in errors[:3]:
            print(f"      • {error}")
    
    return transformed


def get_all_fieldnames(data: List[Dict[str, Any]]) -> List[str]:
    """Собирает все возможные имена полей из данных."""
    fieldnames = set()
    for record in data:
        fieldnames.update(record.keys())
    return sorted(fieldnames)


# ============================================================
# ФАЗА 3: LOAD (ЗАГРУЗКА)
# ============================================================

def load_to_csv(data: List[Dict[str, Any]], output_path: Path) -> None:
    """
    Загружает данные в CSV файл.
    
    Args:
        data: Данные для сохранения
        output_path: Путь к выходному файлу
    """
    print(f"\n💾 LOAD: Сохранение в {output_path}")
    
    if not data:
        print("   ⚠️ Нет данных для сохранения")
        return
    
    # Сбор всех полей
    fieldnames = get_all_fieldnames(data)
    
    # Создание папки, если её нет
    output_path.parent.mkdir(parents=True, exist_ok=True)
    
    # Сохранение
    with open(output_path, 'w', encoding='utf-8', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames, restval='')
        writer.writeheader()
        writer.writerows(data)
    
    print(f"   ✅ Сохранено {len(data)} записей")
    print(f"   📁 Файл: {output_path.absolute()}")
    print(f"   📊 Количество полей: {len(fieldnames)}")
    print(f"   📋 Поля: {', '.join(fieldnames[:10])}" + ('...' if len(fieldnames) > 10 else ''))


# ============================================================
# ОСНОВНАЯ ФУНКЦИЯ
# ============================================================

def main():
    parser = argparse.ArgumentParser(
        description='ETL-скрипт: преобразование JSON в CSV',
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Примеры:
  python json_to_csv.py -i orders.json -o orders.csv
        """
    )
    parser.add_argument('-i', '--input', required=True, help='Входной JSON файл')
    parser.add_argument('-o', '--output', required=True, help='Выходной CSV файл')
    
    args = parser.parse_args()
    
    input_path = Path(args.input)
    output_path = Path(args.output)
    
    print("=" * 60)
    print("JSON → CSV ETL Pipeline")
    print("=" * 60)
    
    try:
        # EXTRACT
        raw_data = extract_from_json(input_path)
        
        if not raw_data:
            print("❌ Нет данных для обработки")
            sys.exit(1)
        
        # TRANSFORM
        transformed_data = transform_order_data(raw_data)
        
        if not transformed_data:
            print("❌ После трансформации не осталось корректных записей")
            sys.exit(1)
        
        # LOAD
        load_to_csv(transformed_data, output_path)
        
        print("\n" + "=" * 60)
        print("✅ ETL ПРОЦЕСС ЗАВЕРШЁН УСПЕШНО!")
        print("=" * 60)
        
    except FileNotFoundError as e:
        print(f"\n❌ Ошибка: {e}")
        sys.exit(1)
    except ValueError as e:
        print(f"\n❌ Ошибка данных: {e}")
        sys.exit(1)
    except Exception as e:
        print(f"\n❌ Непредвиденная ошибка: {e}")
        sys.exit(1)


if __name__ == "__main__":
    main()
Ожидаемый вывод
CSV → JSON:

text
============================================================
CSV → JSON ETL Pipeline
============================================================

📂 EXTRACT: Чтение файла employees.csv
   ✅ Извлечено 5 записей

🔄 TRANSFORM: Обработка данных
   ✅ Успешно обработано: 4 записей
   ⚠️ Ошибок: 1
      • Строка 4: некорректный email ''

💾 LOAD: Сохранение в employees.json
   ✅ Сохранено 4 записей
   📁 Файл: /path/to/employees.json
   📊 Размер: 1,234 байт

============================================================
✅ ETL ПРОЦЕСС ЗАВЕРШЁН УСПЕШНО!
============================================================
JSON → CSV:

text
============================================================
JSON → CSV ETL Pipeline
============================================================

📂 EXTRACT: Чтение файла orders.json
   ✅ Найден массив 'orders' с 2 записями

🔄 TRANSFORM: Обработка данных
   ✅ Успешно обработано: 2 записей

💾 LOAD: Сохранение в orders.csv
   ✅ Сохранено 2 записей
   📁 Файл: /path/to/orders.csv
   📊 Количество полей: 8
   📋 Поля: customer_email, customer_name, items, order_id, processed_at, status, status_ru, total_amount

============================================================
✅ ETL ПРОЦЕСС ЗАВЕРШЁН УСПЕШНО!
============================================================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (прямое преобразование без сложной трансформации)

Варианты 9-17: средний (с трансформацией, валидацией, фильтрацией)

Варианты 18-25: сложный (с вложенными структурами, агрегацией, несколькими источниками)

Варианты 1-8 (Базовый уровень)
№	Тип преобразования	Исходные данные	Особенности
1	CSV → JSON	Список студентов	id, name, age, grade
2	JSON → CSV	Список книг	title, author, year, isbn
3	CSV → JSON	Данные о продуктах	id, name, price, category
4	JSON → CSV	Список городов	name, country, population
5	CSV → JSON	Данные о фильмах	title, director, year, rating
6	JSON → CSV	Список сотрудников	name, position, salary
7	CSV → JSON	Данные о машинах	brand, model, year, price
8	JSON → CSV	Список рецептов	name, cuisine, prep_time
Пример варианта 1:

csv
# students.csv
id,name,age,grade
1,Анна Иванова,20,отлично
2,Борис Петров,19,хорошо
3,Виктор Сидоров,21,удовлетворительно
Варианты 9-17 (Средний уровень)
№	Тип преобразования	Исходные данные	Трансформации
9	CSV → JSON	Транзакции	Вычисление суммы, фильтрация по дате
10	JSON → CSV	Пользователи соцсети	Извлечение вложенных полей (адрес, контакты)
11	CSV → JSON	Данные сенсоров	Группировка по часам, удаление выбросов
12	JSON → CSV	Логи сервера	Парсинг timestamp, фильтрация по уровню
13	CSV → JSON	Оценки студентов	Расчёт среднего балла, ранжирование
14	JSON → CSV	Заказы магазина	Расчёт суммы, группировка по статусу
15	CSV → JSON	Метеоданные	Преобразование единиц измерения
16	JSON → CSV	Комментарии	Очистка текста, удаление спама
17	CSV → JSON	Спортивные результаты	Расчёт рейтинга, определение победителей
Пример варианта 10 (JSON → CSV с вложенными данными):

json
{
    "users": [
        {
            "id": 1,
            "name": "Иван",
            "address": {
                "city": "Москва",
                "street": "Тверская",
                "house": 10
            },
            "contacts": {
                "email": "ivan@example.com",
                "phone": "+71234567890"
            }
        }
    ]
}
Варианты 18-25 (Сложный уровень)
№	Тип преобразования	Исходные данные	Дополнительные требования
18	CSV → JSON	Продажи за несколько лет	Агрегация по месяцам, прогнозирование
19	JSON → CSV	Данные из API (постранично)	Пагинация, rate limiting
20	CSV → JSON	Логи с разных серверов	Объединение, дедупликация
21	JSON → CSV	Вложенные телеметрические данные	Распаковка массивов в строки
22	CSV → JSON	Большой файл (100k+ строк)	Чтение по частям, прогресс-бар
23	JSON → CSV	Данные с ошибками (пропуски, null)	Имputation, логирование
24	CSV → JSON	Конфигурационные файлы	Валидация схемы, проверка типов
25	JSON → CSV	Многомерные данные (тензоры)	Снижение размерности
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	ETL скрипт не работает или работает с критическими ошибками
3 (удовлетворительно)	Базовое преобразование работает, но нет обработки ошибок и валидации
4 (хорошо)	Работают оба преобразования, есть валидация и обработка ошибок
5 (отлично)	Полная реализация с трансформацией, обработкой ошибок, аргументами CLI
5. Шпаргалка
5.1. CSV → JSON (базовый шаблон)
python
import csv
import json

def csv_to_json(input_file, output_file):
    with open(input_file, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        data = list(reader)
    
    with open(output_file, 'w', encoding='utf-8') as f:
        json.dump(data, f, indent=2, ensure_ascii=False)
5.2. JSON → CSV (базовый шаблон)
python
import csv
import json

def json_to_csv(input_file, output_file):
    with open(input_file, 'r', encoding='utf-8') as f:
        data = json.load(f)
    
    if isinstance(data, dict):
        # Пытаемся найти массив
        for key in ['data', 'items', 'results']:
            if key in data:
                data = data[key]
                break
    
    if not data:
        return
    
    with open(output_file, 'w', encoding='utf-8', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=data[0].keys())
        writer.writeheader()
        writer.writerows(data)
5.3. Аргументы командной строки
python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('-i', '--input', required=True)
parser.add_argument('-o', '--output', required=True)
parser.add_argument('--pretty', action='store_true')
args = parser.parse_args()
5.4. Flatten JSON (распаковка вложенных структур)
python
def flatten_json(data, parent='', sep='_'):
    items = {}
    for k, v in data.items():
        new_key = f"{parent}{sep}{k}" if parent else k
        if isinstance(v, dict):
            items.update(flatten_json(v, new_key, sep))
        else:
            items[new_key] = v
    return items
Карточка студента
text
ПЗ 2.17. РАЗРАБОТКА ETL-СКРИПТОВ: CSV → JSON и JSON → CSV

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== ТИП ПРЕОБРАЗОВАНИЯ ===

□ CSV → JSON
□ JSON → CSV

=== ФАЗА EXTRACT ===

□ Проверка существования файла
□ Обработка ошибок чтения
□ Чтение больших файлов

=== ФАЗА TRANSFORM ===

□ Очистка данных
□ Валидация
□ Нормализация
□ Вычисляемые поля
□ Фильтрация

=== ФАЗА LOAD ===

□ Сохранение в файл
□ Форматирование вывода
□ Создание отчёта

=== ОТЧЁТ ===

Файл скрипта: _____________
Входной файл: _____________
Выходной файл: _____________

Дата выполнения: _____________
