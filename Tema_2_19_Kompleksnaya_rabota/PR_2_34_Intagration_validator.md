# ПЗ 2.34. Интеграция валидации в ETL-скрипт

**Тема:** ETL-процессы, валидация данных, Pydantic, обработка ошибок, логирование

**Цель работы:**  
Научиться интегрировать многоуровневую валидацию данных в ETL-скрипты, обрабатывать ошибки валидации, вести логи и отчёты, создавать отказоустойчивые конвейеры обработки данных.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `pydantic`, `pydantic[email]`

```bash
pip install pydantic pydantic[email]
Главная мысль: ETL без валидации — как сборка автомобиля без контроля качества. Интеграция валидации на всех этапах обеспечивает надёжность конечного продукта.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Уровни валидации в ETL
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    УРОВНИ ВАЛИДАЦИИ В ETL                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  УРОВЕНЬ 1: СХЕМА (EXTRACT)                                        │   │
│  │  • Проверка наличия файла/источника                                 │   │
│  │  • Проверка формата данных (CSV, JSON)                              │   │
│  │  • Проверка кодировки                                               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  УРОВЕНЬ 2: СТРУКТУРА (VALIDATE)                                   │   │
│  │  • Проверка наличия обязательных полей                              │   │
│  │  • Проверка типов данных                                            │   │
│  │  • Проверка форматов (email, телефон, дата)                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  УРОВЕНЬ 3: БИЗНЕС-ПРАВИЛА (VALIDATE)                              │   │
│  │  • Проверка диапазонов значений                                      │   │
│  │  • Проверка уникальности                                            │   │
│  │  • Проверка зависимостей между полями                               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  УРОВЕНЬ 4: КАЧЕСТВО (TRANSFORM)                                   │   │
│  │  • Проверка после трансформации                                     │   │
│  │  • Проверка агрегированных данных                                   │   │
│  │  • Проверка целостности                                             │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
1.2. Стратегии обработки ошибок валидации
Стратегия	Описание	Когда использовать
Пропуск (Skip)	Пропустить ошибочную запись	Несколько записей, остальные важны
Останов (Fail)	Прервать выполнение	Ошибка критична для всего набора
Dead Letter Queue	Сохранить ошибки в отдельный файл	Для последующего анализа
Исправление (Fix)	Автоматически исправить ошибку	Простые ошибки (пробелы, регистр)
1.3. Компоненты ETL с валидацией
Компонент	Ответственность
Extractor	Чтение исходных данных, базовая проверка источника
RowValidator	Валидация отдельных записей (Pydantic)
ErrorHandler	Логирование ошибок, отправка в DLQ
Transformer	Преобразование данных
Loader	Загрузка чистых данных
Reporter	Генерация отчёта о валидации
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая интеграцию валидации в ETL-скрипт.

Техническое задание (нулевой вариант)
Разработайте ETL-скрипт для обработки данных о клиентах интернет-магазина. Скрипт должен:

Читать CSV файл с данными клиентов

Валидировать каждую запись с помощью Pydantic

Пропускать невалидные записи и сохранять их в DLQ (Dead Letter Queue)

Трансформировать данные (нормализация, расчёт возраста)

Загружать валидные данные в JSON файл

Генерировать отчёт о валидации

Эталонная реализация
python
#!/usr/bin/env python3
"""
etl_with_validation.py — ETL-скрипт с интеграцией валидации.
"""

import csv
import json
import re
import sys
from pathlib import Path
from datetime import datetime
from typing import List, Dict, Any, Optional, Tuple
from dataclasses import dataclass, field
from enum import Enum

from pydantic import BaseModel, Field, validator, EmailStr, ValidationError


# ============================================================
# PYDANTIC МОДЕЛИ ДЛЯ ВАЛИДАЦИИ
# ============================================================

class CustomerStatus(str, Enum):
    """Статус клиента."""
    ACTIVE = "active"
    INACTIVE = "inactive"
    BLOCKED = "blocked"


class CustomerAddress(BaseModel):
    """Модель адреса клиента."""
    city: str = Field(..., min_length=2, max_length=100)
    street: str = Field(..., min_length=2, max_length=200)
    house: str = Field(..., min_length=1, max_length=10)
    apartment: Optional[str] = Field(None, max_length=10)
    
    @validator('city', 'street')
    def normalize_name(cls, v):
        return v.strip().title()
    
    def get_full_address(self) -> str:
        address = f"{self.city}, {self.street}, {self.house}"
        if self.apartment:
            address += f", кв. {self.apartment}"
        return address


class Customer(BaseModel):
    """Модель клиента для валидации."""
    
    id: int = Field(..., gt=0)
    name: str = Field(..., min_length=2, max_length=100)
    email: EmailStr
    phone: str = Field(..., min_length=10, max_length=15)
    age: int = Field(..., ge=0, le=150)
    status: CustomerStatus = CustomerStatus.ACTIVE
    address: Optional[CustomerAddress] = None
    registered_at: datetime = Field(default_factory=datetime.now)
    
    @validator('name')
    def validate_name(cls, v):
        """Проверка и нормализация имени."""
        if not re.match(r'^[a-zA-Zа-яА-ЯёЁ\s\-]+$', v):
            raise ValueError('Имя может содержать только буквы, пробелы и дефисы')
        return v.strip().title()
    
    @validator('phone')
    def validate_phone(cls, v):
        """Нормализация телефона к формату +7XXXXXXXXXX."""
        digits = re.sub(r'\D', '', v)
        
        if len(digits) == 11 and digits[0] == '8':
            digits = '7' + digits[1:]
        elif len(digits) == 10:
            digits = '7' + digits
        
        if len(digits) != 11 or digits[0] != '7':
            raise ValueError('Неверный формат номера телефона')
        
        return f"+{digits}"
    
    @validator('registered_at', pre=True)
    def parse_date(cls, v):
        """Парсинг даты из строки."""
        if isinstance(v, datetime):
            return v
        
        if isinstance(v, str):
            formats = ["%Y-%m-%d", "%d.%m.%Y", "%Y-%m-%d %H:%M:%S"]
            for fmt in formats:
                try:
                    return datetime.strptime(v, fmt)
                except ValueError:
                    continue
            raise ValueError(f'Неверный формат даты: {v}')
        
        return v


# ============================================================
# ОБРАБОТКА ОШИБОК И DLQ
# ============================================================

@dataclass
class ValidationErrorRecord:
    """Запись об ошибке валидации."""
    row_index: int
    raw_data: Dict
    error_message: str
    timestamp: datetime = field(default_factory=datetime.now)


class DeadLetterQueue:
    """Очередь ошибочных записей (DLQ)."""
    
    def __init__(self, dlq_path: str = "dlq"):
        self.dlq_path = Path(dlq_path)
        self.dlq_path.mkdir(exist_ok=True)
        self.errors: List[ValidationErrorRecord] = []
    
    def add_error(self, row_index: int, raw_data: Dict, error_message: str):
        """Добавление ошибки в DLQ."""
        record = ValidationErrorRecord(
            row_index=row_index,
            raw_data=raw_data,
            error_message=error_message
        )
        self.errors.append(record)
        
        # Немедленная запись в файл
        self._flush()
    
    def _flush(self):
        """Сохранение ошибок в файл."""
        if not self.errors:
            return
        
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        dlq_file = self.dlq_path / f"dlq_{timestamp}.json"
        
        with open(dlq_file, 'w', encoding='utf-8') as f:
            json.dump(
                [{
                    "row_index": e.row_index,
                    "raw_data": e.raw_data,
                    "error_message": e.error_message,
                    "timestamp": e.timestamp.isoformat()
                } for e in self.errors],
                f,
                indent=2,
                ensure_ascii=False
            )
    
    def get_errors(self) -> List[ValidationErrorRecord]:
        """Получение всех ошибок."""
        return self.errors
    
    def get_error_count(self) -> int:
        """Количество ошибок."""
        return len(self.errors)


# ============================================================
# КОМПОНЕНТЫ ETL
# ============================================================

class CSVExtractor:
    """Извлечение данных из CSV файла."""
    
    def __init__(self, filepath: str):
        self.filepath = Path(filepath)
        self.encoding = "utf-8"
    
    def extract(self) -> Tuple[List[Dict], List[Dict]]:
        """
        Извлечение данных из CSV.
        
        Returns:
            (valid_rows, invalid_rows) — данные и строки с ошибками формата
        """
        if not self.filepath.exists():
            raise FileNotFoundError(f"Файл {self.filepath} не найден")
        
        valid_data = []
        invalid_rows = []
        
        try:
            with open(self.filepath, 'r', encoding=self.encoding) as f:
                reader = csv.DictReader(f)
                
                for idx, row in enumerate(reader, start=2):
                    # Преобразование типов
                    if 'id' in row:
                        try:
                            row['id'] = int(row['id'])
                        except ValueError:
                            row['id'] = None
                    
                    if 'age' in row and row['age']:
                        try:
                            row['age'] = int(row['age'])
                        except ValueError:
                            row['age'] = None
                    
                    valid_data.append(row)
                
        except Exception as e:
            print(f"Ошибка чтения CSV: {e}")
            raise
        
        return valid_data, invalid_rows
    
    def validate_source(self) -> bool:
        """Проверка доступности источника."""
        return self.filepath.exists()


class DataValidator:
    """Валидатор данных с использованием Pydantic."""
    
    def __init__(self, model_class: BaseModel, dlq: Optional[DeadLetterQueue] = None):
        self.model_class = model_class
        self.dlq = dlq or DeadLetterQueue()
        self.valid_records = []
        self.invalid_records = []
    
    def validate_row(self, row: Dict, row_index: int) -> Optional[BaseModel]:
        """Валидация одной строки."""
        try:
            validated = self.model_class(**row)
            self.valid_records.append(validated)
            return validated
        except ValidationError as e:
            error_message = "; ".join([f"{err['loc'][0]}: {err['msg']}" for err in e.errors()])
            self.invalid_records.append((row_index, row, error_message))
            self.dlq.add_error(row_index, row, error_message)
            return None
        except Exception as e:
            self.dlq.add_error(row_index, row, str(e))
            return None
    
    def validate_batch(self, data: List[Dict], start_index: int = 2) -> List[BaseModel]:
        """Валидация пакета данных."""
        valid = []
        for idx, row in enumerate(data, start=start_index):
            validated = self.validate_row(row, idx)
            if validated:
                valid.append(validated)
        return valid
    
    def get_statistics(self) -> Dict:
        """Статистика валидации."""
        return {
            "total_valid": len(self.valid_records),
            "total_invalid": len(self.invalid_records),
            "dlq_errors": self.dlq.get_error_count()
        }


class DataTransformer:
    """Трансформация данных."""
    
    def transform(self, data: List[Customer]) -> List[Dict]:
        """
        Трансформация валидных данных.
        
        Операции:
        - Расчёт возраста (если нет, по дате регистрации)
        - Добавление вычисляемых полей
        - Форматирование адреса
        """
        transformed = []
        
        for customer in data:
            # Базовые поля
            transformed_customer = {
                "id": customer.id,
                "name": customer.name,
                "email": customer.email,
                "phone": customer.phone,
                "age": customer.age,
                "status": customer.status.value,
                "registered_at": customer.registered_at.isoformat()
            }
            
            # Расчёт возраста по дате регистрации (если age не указан)
            if not customer.age and customer.registered_at:
                age = datetime.now().year - customer.registered_at.year
                transformed_customer["calculated_age"] = age
            
            # Возрастная категория
            age_val = customer.age or transformed_customer.get("calculated_age", 0)
            if age_val < 18:
                transformed_customer["age_category"] = "minor"
            elif age_val < 65:
                transformed_customer["age_category"] = "adult"
            else:
                transformed_customer["age_category"] = "senior"
            
            # Адрес
            if customer.address:
                transformed_customer["address"] = customer.address.get_full_address()
            else:
                transformed_customer["address"] = None
            
            # Временная метка обработки
            transformed_customer["processed_at"] = datetime.now().isoformat()
            
            transformed.append(transformed_customer)
        
        return transformed


class JSONLoader:
    """Загрузка данных в JSON файл."""
    
    def __init__(self, output_path: str):
        self.output_path = Path(output_path)
        self.output_path.parent.mkdir(parents=True, exist_ok=True)
    
    def load(self, data: List[Dict]) -> bool:
        """Загрузка данных в JSON."""
        try:
            with open(self.output_path, 'w', encoding='utf-8') as f:
                json.dump(data, f, indent=2, ensure_ascii=False, default=str)
            return True
        except Exception as e:
            print(f"Ошибка загрузки: {e}")
            return False


class ValidationReporter:
    """Генератор отчётов о валидации."""
    
    def __init__(self, report_path: str = "validation_report.json"):
        self.report_path = Path(report_path)
    
    def generate_report(
        self,
        extract_stats: Dict,
        validation_stats: Dict,
        transform_stats: Dict,
        load_stats: Dict,
        start_time: datetime,
        end_time: datetime
    ) -> Dict:
        """Генерация отчёта."""
        report = {
            "timestamp": datetime.now().isoformat(),
            "duration_seconds": (end_time - start_time).total_seconds(),
            "extract": extract_stats,
            "validation": validation_stats,
            "transform": transform_stats,
            "load": load_stats,
            "summary": {
                "total_processed": extract_stats.get("total_rows", 0),
                "valid_records": validation_stats.get("total_valid", 0),
                "invalid_records": validation_stats.get("total_invalid", 0),
                "success_rate": f"{validation_stats.get('total_valid', 0) / max(extract_stats.get('total_rows', 1), 1) * 100:.1f}%"
            }
        }
        
        # Сохранение отчёта
        with open(self.report_path, 'w', encoding='utf-8') as f:
            json.dump(report, f, indent=2, ensure_ascii=False)
        
        return report
    
    def print_summary(self, report: Dict):
        """Вывод сводки в консоль."""
        print("\n" + "=" * 60)
        print("ОТЧЁТ О ВАЛИДАЦИИ")
        print("=" * 60)
        
        summary = report.get("summary", {})
        print(f"\n📊 СТАТИСТИКА:")
        print(f"   Всего записей: {summary.get('total_processed', 0)}")
        print(f"   ✅ Валидных: {summary.get('valid_records', 0)}")
        print(f"   ❌ Невалидных: {summary.get('invalid_records', 0)}")
        print(f"   📈 Успешность: {summary.get('success_rate', '0%')}")
        
        print(f"\n⏱️ Время выполнения: {report.get('duration_seconds', 0):.2f} сек")
        print(f"📁 Отчёт сохранён: {self.report_path}")


# ============================================================
# ETL ПАЙПЛАЙН
# ============================================================

class ETLPipeline:
    """Основной ETL-пайплайн с валидацией."""
    
    def __init__(
        self,
        input_file: str,
        output_file: str,
        model_class: BaseModel = Customer,
        dlq_path: str = "dlq",
        report_path: str = "validation_report.json"
    ):
        self.input_file = input_file
        self.output_file = output_file
        
        self.extractor = CSVExtractor(input_file)
        self.dlq = DeadLetterQueue(dlq_path)
        self.validator = DataValidator(model_class, self.dlq)
        self.transformer = DataTransformer()
        self.loader = JSONLoader(output_file)
        self.reporter = ValidationReporter(report_path)
        
        self.extract_stats = {}
        self.validation_stats = {}
        self.transform_stats = {}
        self.load_stats = {}
        self.start_time = None
        self.end_time = None
    
    def run(self) -> bool:
        """Запуск ETL-пайплайна."""
        self.start_time = datetime.now()
        
        print("=" * 60)
        print("🚀 ЗАПУСК ETL-ПАЙПЛАЙНА С ВАЛИДАЦИЕЙ")
        print("=" * 60)
        
        try:
            # STEP 1: EXTRACT
            print("\n📂 1. ИЗВЛЕЧЕНИЕ ДАННЫХ")
            print("-" * 40)
            
            if not self.extractor.validate_source():
                print("❌ Источник данных недоступен")
                return False
            
            raw_data, invalid_csv = self.extractor.extract()
            self.extract_stats = {
                "total_rows": len(raw_data),
                "csv_errors": len(invalid_csv),
                "source": self.input_file
            }
            print(f"   Прочитано записей: {len(raw_data)}")
            print(f"   Ошибок CSV: {len(invalid_csv)}")
            
            # STEP 2: VALIDATE
            print("\n🔍 2. ВАЛИДАЦИЯ ДАННЫХ")
            print("-" * 40)
            
            valid_data = self.validator.validate_batch(raw_data, start_index=2)
            self.validation_stats = self.validator.get_statistics()
            print(f"   ✅ Валидных: {self.validation_stats['total_valid']}")
            print(f"   ❌ Невалидных: {self.validation_stats['total_invalid']}")
            print(f"   📦 DLQ записей: {self.validation_stats['dlq_errors']}")
            
            if not valid_data:
                print("   ⚠️ Нет валидных данных для обработки")
            
            # STEP 3: TRANSFORM
            print("\n🔄 3. ТРАНСФОРМАЦИЯ ДАННЫХ")
            print("-" * 40)
            
            transformed_data = self.transformer.transform(valid_data)
            self.transform_stats = {
                "transformed_count": len(transformed_data),
                "operations": ["normalization", "age_calculation", "enrichment"]
            }
            print(f"   Трансформировано записей: {len(transformed_data)}")
            
            # STEP 4: LOAD
            print("\n💾 4. ЗАГРУЗКА ДАННЫХ")
            print("-" * 40)
            
            success = self.loader.load(transformed_data)
            self.load_stats = {
                "loaded_count": len(transformed_data) if success else 0,
                "destination": self.output_file,
                "success": success
            }
            
            if success:
                print(f"   ✅ Загружено записей: {len(transformed_data)}")
                print(f"   📁 Файл: {self.output_file}")
            else:
                print(f"   ❌ Ошибка загрузки")
            
            self.end_time = datetime.now()
            
            # STEP 5: REPORT
            print("\n📊 5. ГЕНЕРАЦИЯ ОТЧЁТА")
            print("-" * 40)
            
            report = self.reporter.generate_report(
                extract_stats=self.extract_stats,
                validation_stats=self.validation_stats,
                transform_stats=self.transform_stats,
                load_stats=self.load_stats,
                start_time=self.start_time,
                end_time=self.end_time
            )
            
            self.reporter.print_summary(report)
            
            return success
            
        except Exception as e:
            self.end_time = datetime.now()
            print(f"\n❌ КРИТИЧЕСКАЯ ОШИБКА: {e}")
            import traceback
            traceback.print_exc()
            return False


# ============================================================
# СОЗДАНИЕ ТЕСТОВЫХ ДАННЫХ
# ============================================================

def create_test_data():
    """Создание тестового CSV файла."""
    test_csv = Path("test_customers.csv")
    
    with open(test_csv, 'w', encoding='utf-8', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(["id", "name", "email", "phone", "age", "status", "address_city", "address_street", "address_house"])
        
        # Валидные записи
        writer.writerow([1, "иван петров", "ivan@example.com", "+79123456789", 25, "active", "Москва", "Тверская", "10"])
        writer.writerow([2, "мария сидорова", "maria@example.com", "89231234567", 30, "active", "СПб", "Невский", "25", "42"])
        
        # Невалидные записи
        writer.writerow([3, "", "invalid-email", "123", -5, "unknown", "", "", ""])  # пустое имя, неверный email
        writer.writerow([4, "Пётр", "petr@example.com", "+7", 200, "active", "Москва", "", ""])  # неверный возраст
    
    return test_csv


# ============================================================
# ЗАПУСК
# ============================================================

def main():
    """Главная функция."""
    # Создание тестовых данных
    test_file = create_test_data()
    
    # Запуск ETL
    pipeline = ETLPipeline(
        input_file=str(test_file),
        output_file="output/customers.json",
        dlq_path="dlq"
    )
    
    success = pipeline.run()
    
    print("\n" + "=" * 60)
    print("✅ ETL-ПРОЦЕСС ЗАВЕРШЁН" if success else "❌ ETL-ПРОЦЕСС НЕ УДАЛСЯ")
    print("=" * 60)
    
    return 0 if success else 1


if __name__ == "__main__":
    sys.exit(main())
Ожидаемый вывод
text
============================================================
🚀 ЗАПУСК ETL-ПАЙПЛАЙНА С ВАЛИДАЦИЕЙ
============================================================

📂 1. ИЗВЛЕЧЕНИЕ ДАННЫХ
----------------------------------------
   Прочитано записей: 4
   Ошибок CSV: 0

🔍 2. ВАЛИДАЦИЯ ДАННЫХ
----------------------------------------
   ✅ Валидных: 2
   ❌ Невалидных: 2
   📦 DLQ записей: 2

🔄 3. ТРАНСФОРМАЦИЯ ДАННЫХ
----------------------------------------
   Трансформировано записей: 2

💾 4. ЗАГРУЗКА ДАННЫХ
----------------------------------------
   ✅ Загружено записей: 2
   📁 Файл: output/customers.json

📊 5. ГЕНЕРАЦИЯ ОТЧЁТА
----------------------------------------

============================================================
ОТЧЁТ О ВАЛИДАЦИИ
============================================================

📊 СТАТИСТИКА:
   Всего записей: 4
   ✅ Валидных: 2
   ❌ Невалидных: 2
   📈 Успешность: 50.0%

⏱️ Время выполнения: 0.15 сек
📁 Отчёт сохранён: validation_report.json

============================================================
✅ ETL-ПРОЦЕСС ЗАВЕРШЁН
============================================================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (один источник, простая валидация)

Варианты 9-17: средний (несколько источников, сложная валидация)

Варианты 18-25: сложный (распределённая обработка, реальные API)

Варианты 1-8 (Базовый уровень)
№	Источник	Цель	Валидация
1	CSV (студенты)	JSON	name, age, grade
2	CSV (товары)	JSON	name, price, stock
3	CSV (книги)	JSON	title, author, year
4	CSV (фильмы)	JSON	title, director, rating
5	CSV (сотрудники)	JSON	name, position, salary
6	CSV (заказы)	JSON	order_id, total, status
7	CSV (пользователи)	JSON	username, email, age
8	CSV (события)	JSON	event_name, date, location
Варианты 9-17 (Средний уровень)
№	Источник	Цель	Дополнительная валидация
9	CSV (клиенты)	JSON + DLQ	email, phone, address
10	CSV + API	JSON	Внешняя проверка email
11	CSV (транзакции)	JSON + отчёт	Сумма, валюта, лимиты
12	JSON (логи)	CSV	Парсинг дат, IP
13	CSV (продажи)	JSON + агрегация	Цены, скидки, итоги
14	CSV (отели)	JSON	Даты, количество гостей
15	CSV (авиабилеты)	JSON	Маршрут, пассажиры
16	CSV (рецепты)	JSON	Ингредиенты, время
17	CSV (отзывы)	JSON	Рейтинг, текст
Варианты 18-25 (Сложный уровень)
№	Тема	Особенности
18	Банковские транзакции	CSV → БД, алгоритм Луна, лимиты
19	Медицинские данные	CSV → JSON, валидация СНИЛС, ОМС
20	Налоговые отчёты	CSV → JSON, ИНН, КПП, расчёты
21	Логи сервера	TXT → JSON, парсинг, агрегация
22	Данные сенсоров	CSV → БД, временные ряды
23	Социальные сети	JSON → CSV, пользователи, посты
24	Финансовые отчёты	CSV → Excel, бюджеты, факты
25	E-commerce	CSV + JSON → БД, заказы, товары, клиенты
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	ETL не работает или валидация отсутствует
3 (удовлетворительно)	Базовый ETL с валидацией типов
4 (хорошо)	+ DLQ, отчёты, трансформация
5 (отлично)	+ многоуровневая валидация, бизнес-правила, мониторинг
5. Шпаргалка
python
# === ОСНОВНЫЕ КОМПОНЕНТЫ ===
extractor = CSVExtractor("input.csv")
validator = DataValidator(CustomerModel)
transformer = DataTransformer()
loader = JSONLoader("output.json")
dlq = DeadLetterQueue("dlq")

# === ЗАПУСК ПАЙПЛАЙНА ===
pipeline = ETLPipeline("input.csv", "output.json")
success = pipeline.run()

# === ВАЛИДАЦИЯ СТРОКИ ===
@validator('field')
def validate_field(cls, v):
    if not condition:
        raise ValueError('Ошибка')
    return v

# === DLQ ===
dlq.add_error(row_index, row, error_message)

# === ОТЧЁТ ===
reporter = ValidationReporter()
report = reporter.generate_report(...)
Карточка студента
text
ПЗ 2.34. ИНТЕГРАЦИЯ ВАЛИДАЦИИ В ETL-СКРИПТ

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== КОМПОНЕНТЫ ===

□ Extractor (источник: _____________)
□ Validator (модель: _____________)
□ Transformer (операции: _____________)
□ Loader (назначение: _____________)
□ DLQ (dead letter queue)
□ Reporter (отчёты)

=== ВАЛИДАЦИЯ ===

□ Схема данных (обязательные поля)
□ Типы данных
□ Форматы (email, phone, date)
□ Диапазоны (age, price)
□ Бизнес-правила
□ Межполевая валидация

=== ОТЧЁТ ===

Код ETL: _____________
Результат выполнения: _____________
DLQ записи: _____________
Отчёт о валидации: _____________

Дата выполнения: _____________
