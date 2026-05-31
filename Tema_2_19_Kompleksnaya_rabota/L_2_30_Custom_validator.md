# Тема 2.30. Кастомные валидаторы. Валидация между полями. Интеграция валидации в ETL-процесс

**Цель лекции:**  
Научиться создавать кастомные валидаторы для специфических бизнес-правил, реализовывать валидацию зависимостей между полями, интегрировать многоуровневую валидацию в ETL-процесс.

> Главная мысль: **Стандартные валидаторы проверяют отдельные поля. Кастомные валидаторы проверяют бизнес-логику. Вместе они создают надёжную защиту данных.**

---

## Содержание

1. [Введение в кастомные валидаторы](#1-введение-в-кастомные-валидаторы)
2. [Валидация между полями](#2-валидация-между-полями)
3. [Иерархия валидаторов](#3-иерархия-валидаторов)
4. [Интеграция валидации в ETL-процесс](#4-интеграция-валидации-в-etl-процесс)
5. [Практические примеры](#5-практические-примеры)
6. [Контрольные вопросы](#6-контрольные-вопросы)
7. [Практическое задание](#7-практическое-задание)
8. [Шпаргалка](#8-шпаргалка)

---

## 1. Введение в кастомные валидаторы

### 1.1. Когда нужны кастомные валидаторы

| Ситуация | Пример | Стандартный валидатор | Кастомный валидатор |
|----------|--------|----------------------|---------------------|
| Сложные форматы | Номер паспорта, ИНН | Частично | ✅ |
| Бизнес-правила | Сумма заказа не может превышать лимит | ❌ | ✅ |
| Зависимые поля | `start_date < end_date` | ❌ | ✅ |
| Внешние проверки | Email должен быть уникальным в БД | ❌ | ✅ |
| Контекстные правила | Скидка зависит от суммы заказа | ❌ | ✅ |

### 1.2. Базовый класс кастомного валидатора

```python
from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional
from dataclasses import dataclass
from datetime import datetime


@dataclass
class ValidationError:
    """Ошибка валидации."""
    row_index: int
    field: str
    value: Any
    message: str
    error_type: str
    severity: str = "error"  # error, warning, info


class CustomValidator(ABC):
    """Базовый класс для кастомных валидаторов."""
    
    def __init__(self, config: Optional[Dict] = None):
        self.config = config or {}
        self.errors: List[ValidationError] = []
        self.warnings: List[ValidationError] = []
    
    @abstractmethod
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        """
        Валидация данных.
        
        Returns:
            Валидные данные (ошибочные записи удаляются или помечаются)
        """
        pass
    
    def add_error(self, row_index: int, field: str, value: Any, message: str):
        """Добавление ошибки."""
        self.errors.append(ValidationError(
            row_index=row_index,
            field=field,
            value=value,
            message=message,
            error_type=self.__class__.__name__
        ))
    
    def add_warning(self, row_index: int, field: str, value: Any, message: str):
        """Добавление предупреждения."""
        self.warnings.append(ValidationError(
            row_index=row_index,
            field=field,
            value=value,
            message=message,
            error_type=self.__class__.__name__,
            severity="warning"
        ))
    
    def get_errors(self) -> List[ValidationError]:
        """Получение списка ошибок."""
        return self.errors
    
    def get_warnings(self) -> List[ValidationError]:
        """Получение списка предупреждений."""
        return self.warnings
    
    def has_errors(self) -> bool:
        """Есть ли ошибки."""
        return len(self.errors) > 0
    
    def clear(self):
        """Очистка ошибок и предупреждений."""
        self.errors.clear()
        self.warnings.clear()
2. Валидация между полями
2.1. Примеры зависимостей между полями
Тип зависимости	Пример	Ошибка
Временная	start_date < end_date	"Дата начала позже даты окончания"
Количественная	discount <= price	"Скидка не может превышать цену"
Логическая	is_student == True AND age < 25	"Возраст студента не может быть больше 25"
Условная	country == "USA" THEN zipcode MATCHES regex	"Неверный формат ZIP-кода для США"
Агрегатная	sum(items.price) == total	"Сумма товаров не совпадает с общей"
2.2. Валидатор диапазона дат
python
from datetime import datetime
from typing import Dict, Any, List, Optional


class DateRangeValidator(CustomValidator):
    """
    Валидатор диапазона дат.
    Проверяет, что start_date < end_date.
    """
    
    def __init__(
        self,
        start_field: str,
        end_field: str,
        allow_equal: bool = False,
        config: Optional[Dict] = None
    ):
        super().__init__(config)
        self.start_field = start_field
        self.end_field = end_field
        self.allow_equal = allow_equal
    
    def _parse_date(self, value: Any) -> Optional[datetime]:
        """Парсинг даты из различных форматов."""
        if isinstance(value, datetime):
            return value
        
        if isinstance(value, str):
            formats = [
                "%Y-%m-%d",
                "%Y-%m-%d %H:%M:%S",
                "%d.%m.%Y",
                "%d/%m/%Y",
                "%Y%m%d"
            ]
            for fmt in formats:
                try:
                    return datetime.strptime(value, fmt)
                except ValueError:
                    continue
        
        return None
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = []
        
        for idx, row in enumerate(data):
            start_value = row.get(self.start_field)
            end_value = row.get(self.end_field)
            
            # Пропускаем, если поля отсутствуют
            if start_value is None or end_value is None:
                valid_data.append(row)
                continue
            
            start_date = self._parse_date(start_value)
            end_date = self._parse_date(end_value)
            
            if start_date is None:
                self.add_error(
                    idx, self.start_field, start_value,
                    f"Неверный формат даты: {start_value}"
                )
                continue
            
            if end_date is None:
                self.add_error(
                    idx, self.end_field, end_value,
                    f"Неверный формат даты: {end_value}"
                )
                continue
            
            if self.allow_equal:
                is_valid = start_date <= end_date
                error_msg = f"Дата начала ({start_date}) должна быть не позже даты окончания ({end_date})"
            else:
                is_valid = start_date < end_date
                error_msg = f"Дата начала ({start_date}) должна быть раньше даты окончания ({end_date})"
            
            if not is_valid:
                self.add_error(idx, f"{self.start_field},{self.end_field}", 
                              f"{start_date} -> {end_date}", error_msg)
                continue
            
            valid_data.append(row)
        
        return valid_data
2.3. Валидатор суммы
python
class SumValidator(CustomValidator):
    """
    Валидатор суммы.
    Проверяет, что сумма полей равна указанному значению.
    """
    
    def __init__(
        self,
        fields: List[str],
        total_field: str,
        tolerance: float = 0.01,
        config: Optional[Dict] = None
    ):
        super().__init__(config)
        self.fields = fields
        self.total_field = total_field
        self.tolerance = tolerance
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = []
        
        for idx, row in enumerate(data):
            # Суммируем указанные поля
            try:
                total = sum(float(row.get(field, 0)) for field in self.fields)
            except (ValueError, TypeError):
                self.add_error(
                    idx, ",".join(self.fields), row,
                    "Некорректные числовые значения для суммирования"
                )
                continue
            
            expected = row.get(self.total_field)
            
            if expected is None:
                self.add_error(
                    idx, self.total_field, None,
                    f"Поле {self.total_field} отсутствует"
                )
                continue
            
            try:
                expected = float(expected)
            except (ValueError, TypeError):
                self.add_error(
                    idx, self.total_field, expected,
                    f"Поле {self.total_field} должно быть числом"
                )
                continue
            
            if abs(total - expected) > self.tolerance:
                self.add_error(
                    idx, self.total_field, expected,
                    f"Сумма полей {total} не равна ожидаемой {expected}"
                )
                continue
            
            # Добавляем вычисленную сумму для использования в трансформации
            row['_calculated_sum'] = total
            valid_data.append(row)
        
        return valid_data
2.4. Условный валидатор
python
class ConditionalValidator(CustomValidator):
    """
    Условный валидатор.
    Применяет валидацию только если выполняется условие.
    """
    
    def __init__(
        self,
        condition: Dict[str, Any],
        validator: CustomValidator,
        config: Optional[Dict] = None
    ):
        super().__init__(config)
        self.condition_field = condition.get("field")
        self.condition_value = condition.get("value")
        self.condition_operator = condition.get("operator", "eq")
        self.validator = validator
    
    def _check_condition(self, row: Dict[str, Any]) -> bool:
        """Проверка условия."""
        if self.condition_field is None:
            return True
        
        value = row.get(self.condition_field)
        
        if self.condition_operator == "eq":
            return value == self.condition_value
        elif self.condition_operator == "ne":
            return value != self.condition_value
        elif self.condition_operator == "in":
            return value in self.condition_value
        elif self.condition_operator == "not_in":
            return value not in self.condition_value
        elif self.condition_operator == "gt":
            return value > self.condition_value
        elif self.condition_operator == "lt":
            return value < self.condition_value
        
        return False
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        # Разделяем данные на те, что нужно валидировать, и остальные
        to_validate = []
        to_skip = []
        
        for idx, row in enumerate(data):
            if self._check_condition(row):
                to_validate.append((idx, row))
            else:
                to_skip.append(row)
        
        # Валидируем только те записи, которые подходят под условие
        if to_validate:
            indices, rows = zip(*to_validate)
            validated = self.validator.validate(list(rows))
            
            # Переносим ошибки из вложенного валидатора
            self.errors.extend(self.validator.get_errors())
            self.warnings.extend(self.validator.get_warnings())
            
            # Объединяем результаты
            return list(validated) + to_skip
        
        return data
3. Иерархия валидаторов
3.1. Композитный валидатор с уровнями
python
class ValidationLevel:
    """Уровни валидации."""
    SCHEMA = 1      # Проверка структуры
    TYPE = 2        # Проверка типов
    FORMAT = 3      # Проверка формата
    BUSINESS = 4    # Бизнес-правила
    CROSS = 5       # Межполевая валидация
    EXTERNAL = 6    # Внешние проверки


class HierarchicalValidator(CustomValidator):
    """
    Иерархический валидатор.
    Выполняет валидацию в определённом порядке и может
    прекращать выполнение при ошибках на критических уровнях.
    """
    
    def __init__(
        self,
        validators: Dict[int, List[CustomValidator]],
        stop_on_level: int = ValidationLevel.BUSINESS,
        config: Optional[Dict] = None
    ):
        super().__init__(config)
        self.validators = validators
        self.stop_on_level = stop_on_level
        self.level_results: Dict[int, bool] = {}
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        current_data = data
        
        for level in sorted(self.validators.keys()):
            self.level_results[level] = True
            
            for validator in self.validators[level]:
                current_data = validator.validate(current_data)
                self.errors.extend(validator.get_errors())
                self.warnings.extend(validator.get_warnings())
                
                if validator.has_errors() and level >= self.stop_on_level:
                    self.level_results[level] = False
                    return current_data
        
        return current_data
    
    def get_report(self) -> Dict:
        """Отчёт о прохождении уровней валидации."""
        return {
            "levels": self.level_results,
            "total_errors": len(self.errors),
            "total_warnings": len(self.warnings),
            "passed": all(self.level_results.values())
        }
3.2. Валидатор с исправлением ошибок
python
class CorrectiveValidator(CustomValidator):
    """
    Корректирующий валидатор.
    Не только обнаруживает ошибки, но и исправляет их.
    """
    
    def __init__(self, config: Optional[Dict] = None):
        super().__init__(config)
        self.fixed_count = 0
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = []
        self.fixed_count = 0
        
        for idx, row in enumerate(data):
            row, fixed = self._fix_row(row, idx)
            if fixed:
                self.fixed_count += 1
            valid_data.append(row)
        
        return valid_data
    
    def _fix_row(self, row: Dict, idx: int) -> tuple:
        """Исправление строки. Переопределяется в наследниках."""
        return row, False
    
    def get_fixed_count(self) -> int:
        return self.fixed_count


class EmailNormalizer(CorrectiveValidator):
    """Нормализация email-адресов."""
    
    def _fix_row(self, row: Dict, idx: int) -> tuple:
        fixed = False
        email = row.get('email')
        
        if email and isinstance(email, str):
            original = email
            email = email.lower().strip()
            
            if email != original:
                row['email'] = email
                fixed = True
                self.add_warning(idx, 'email', original, f"Email нормализован: {original} -> {email}")
        
        return row, fixed


class PhoneNormalizer(CorrectiveValidator):
    """Нормализация телефонных номеров."""
    
    def _fix_row(self, row: Dict, idx: int) -> tuple:
        fixed = False
        phone = row.get('phone')
        
        if phone and isinstance(phone, str):
            import re
            original = phone
            
            # Удаляем все нецифровые символы
            digits = re.sub(r'\D', '', phone)
            
            # Приводим к формату +7XXXXXXXXXX
            if len(digits) == 11 and digits[0] == '8':
                digits = '7' + digits[1:]
                row['phone'] = f"+{digits}"
                fixed = True
            elif len(digits) == 10:
                row['phone'] = f"+7{digits}"
                fixed = True
            
            if fixed:
                self.add_warning(idx, 'phone', original, f"Телефон нормализован: {original} -> {row['phone']}")
        
        return row, fixed
4. Интеграция валидации в ETL-процесс
4.1. ETL-пайплайн с многоуровневой валидацией
python
from datetime import datetime
from typing import List, Dict, Any, Optional
from dataclasses import dataclass


@dataclass
class ETLReport:
    """Отчёт о выполнении ETL-процесса."""
    started_at: datetime
    finished_at: datetime
    extracted_count: int
    validated_count: int
    transformed_count: int
    loaded_count: int
    errors: List[Dict]
    warnings: List[Dict]
    success: bool


class ETLPipelineWithValidation:
    """ETL-пайплайн с интегрированной валидацией."""
    
    def __init__(
        self,
        name: str,
        extractor,
        validators: List[CustomValidator],
        transformers: List,
        loader,
        error_handler=None
    ):
        self.name = name
        self.extractor = extractor
        self.validators = validators
        self.transformers = transformers
        self.loader = loader
        self.error_handler = error_handler
        self.logger = None  # упрощённо
    
    def _run_validation(self, data: List[Dict]) -> List[Dict]:
        """Запуск всех валидаторов."""
        valid_data = data
        
        for validator in self.validators:
            validator.clear()
            valid_data = validator.validate(valid_data)
            
            # Логирование ошибок и предупреждений
            for error in validator.get_errors():
                self.logger.warning(f"Ошибка: {error}")
            
            for warning in validator.get_warnings():
                self.logger.info(f"Предупреждение: {warning}")
        
        return valid_data
    
    def run(self) -> ETLReport:
        """Запуск ETL-процесса с валидацией."""
        started_at = datetime.now()
        
        try:
            # EXTRACT
            self.logger.info("Извлечение данных...")
            raw_data = self.extractor.extract()
            extracted_count = len(raw_data)
            
            # VALIDATE
            self.logger.info("Валидация данных...")
            validated_data = self._run_validation(raw_data)
            validated_count = len(validated_data)
            
            # TRANSFORM
            self.logger.info("Трансформация данных...")
            transformed_data = validated_data
            for transformer in self.transformers:
                transformed_data = transformer.transform(transformed_data)
            transformed_count = len(transformed_data)
            
            # LOAD
            self.logger.info("Загрузка данных...")
            self.loader.load(transformed_data)
            loaded_count = len(transformed_data)
            
            # Сбор ошибок
            errors = []
            warnings = []
            for validator in self.validators:
                errors.extend(validator.get_errors())
                warnings.extend(validator.get_warnings())
            
            finished_at = datetime.now()
            
            return ETLReport(
                started_at=started_at,
                finished_at=finished_at,
                extracted_count=extracted_count,
                validated_count=validated_count,
                transformed_count=transformed_count,
                loaded_count=loaded_count,
                errors=[e.__dict__ for e in errors],
                warnings=[w.__dict__ for w in warnings],
                success=True
            )
            
        except Exception as e:
            finished_at = datetime.now()
            
            if self.error_handler:
                self.error_handler.handle(e)
            
            return ETLReport(
                started_at=started_at,
                finished_at=finished_at,
                extracted_count=0,
                validated_count=0,
                transformed_count=0,
                loaded_count=0,
                errors=[{"message": str(e)}],
                warnings=[],
                success=False
            )
4.2. Отчёт о валидации
python
import json
from pathlib import Path


class ValidationReporter:
    """Генератор отчётов о валидации."""
    
    def __init__(self, report_dir: str = "validation_reports"):
        self.report_dir = Path(report_dir)
        self.report_dir.mkdir(exist_ok=True)
    
    def generate_report(self, validators: List[CustomValidator], etl_report: ETLReport) -> Dict:
        """Генерация отчёта."""
        report = {
            "timestamp": datetime.now().isoformat(),
            "etl_summary": {
                "extracted": etl_report.extracted_count,
                "validated": etl_report.validated_count,
                "transformed": etl_report.transformed_count,
                "loaded": etl_report.loaded_count,
                "success": etl_report.success
            },
            "validation": {
                "total_errors": len(etl_report.errors),
                "total_warnings": len(etl_report.warnings),
                "errors_by_type": self._group_by_type(etl_report.errors),
                "warnings_by_type": self._group_by_type(etl_report.warnings),
                "errors_by_field": self._group_by_field(etl_report.errors),
                "sample_errors": etl_report.errors[:10]
            }
        }
        
        # Сохранение в файл
        report_file = self.report_dir / f"validation_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
        with open(report_file, 'w', encoding='utf-8') as f:
            json.dump(report, f, indent=2, ensure_ascii=False)
        
        return report
    
    def _group_by_type(self, errors: List[Dict]) -> Dict[str, int]:
        """Группировка ошибок по типу."""
        result = {}
        for error in errors:
            error_type = error.get('error_type', 'unknown')
            result[error_type] = result.get(error_type, 0) + 1
        return result
    
    def _group_by_field(self, errors: List[Dict]) -> Dict[str, int]:
        """Группировка ошибок по полю."""
        result = {}
        for error in errors:
            field = error.get('field', 'unknown')
            result[field] = result.get(field, 0) + 1
        return result
    
    def print_summary(self, report: Dict):
        """Вывод сводки в консоль."""
        print("\n" + "=" * 60)
        print("ОТЧЁТ О ВАЛИДАЦИИ ДАННЫХ")
        print("=" * 60)
        
        summary = report["etl_summary"]
        print(f"\n📊 СТАТИСТИКА ОБРАБОТКИ:")
        print(f"   Извлечено: {summary['extracted']}")
        print(f"   После валидации: {summary['validated']}")
        print(f"   После трансформации: {summary['transformed']}")
        print(f"   Загружено: {summary['loaded']}")
        
        validation = report["validation"]
        print(f"\n🔍 ОШИБКИ ВАЛИДАЦИИ:")
        print(f"   Всего ошибок: {validation['total_errors']}")
        print(f"   Всего предупреждений: {validation['total_warnings']}")
        
        if validation['errors_by_type']:
            print(f"\n   Распределение по типам:")
            for error_type, count in validation['errors_by_type'].items():
                print(f"      - {error_type}: {count}")
        
        if validation['errors_by_field']:
            print(f"\n   Распределение по полям:")
            for field, count in list(validation['errors_by_field'].items())[:5]:
                print(f"      - {field}: {count}")
5. Практические примеры
5.1. Комплексный пример: валидация заказов
python
# order_validation.py
"""
Комплексная валидация заказов с использованием кастомных валидаторов.
"""

import json
from datetime import datetime
from typing import Dict, Any, List, Optional
from decimal import Decimal


# ============================================================
# КАСТОМНЫЕ ВАЛИДАТОРЫ ДЛЯ ЗАКАЗОВ
# ============================================================

class OrderDateValidator(CustomValidator):
    """Валидация дат заказа."""
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = []
        now = datetime.now()
        
        for idx, row in enumerate(data):
            order_date = row.get('order_date')
            delivery_date = row.get('delivery_date')
            
            # Проверка, что дата заказа не в будущем
            if order_date:
                if order_date > now:
                    self.add_error(idx, 'order_date', order_date, 
                                  "Дата заказа не может быть в будущем")
                    continue
            
            # Проверка, что дата доставки позже даты заказа
            if order_date and delivery_date:
                if delivery_date < order_date:
                    self.add_error(idx, 'delivery_date', delivery_date,
                                  "Дата доставки не может быть раньше даты заказа")
                    continue
            
            valid_data.append(row)
        
        return valid_data


class PaymentValidator(CustomValidator):
    """Валидация платежей."""
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = []
        
        for idx, row in enumerate(data):
            payment_method = row.get('payment_method')
            card_number = row.get('card_number')
            card_expiry = row.get('card_expiry')
            
            # Если метод оплаты "card", проверяем данные карты
            if payment_method == 'card':
                if not card_number:
                    self.add_error(idx, 'card_number', None,
                                  "Для оплаты картой требуется номер карты")
                    continue
                
                if not card_expiry:
                    self.add_error(idx, 'card_expiry', None,
                                  "Для оплаты картой требуется срок действия")
                    continue
                
                # Простая проверка длины номера карты
                if len(str(card_number)) not in [15, 16]:
                    self.add_warning(idx, 'card_number', card_number,
                                    "Нестандартная длина номера карты")
            
            valid_data.append(row)
        
        return valid_data


class DiscountValidator(CustomValidator):
    """Валидация скидок (межполевая)."""
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = []
        
        for idx, row in enumerate(data):
            discount = row.get('discount', 0)
            total = row.get('total', 0)
            
            # Скидка не может быть больше общей суммы
            if discount > total:
                self.add_error(idx, 'discount', discount,
                              f"Скидка {discount} не может превышать сумму заказа {total}")
                continue
            
            # Скидка не может быть отрицательной
            if discount < 0:
                self.add_error(idx, 'discount', discount,
                              "Скидка не может быть отрицательной")
                continue
            
            # Предупреждение о большой скидке
            if discount > total * 0.5:
                self.add_warning(idx, 'discount', discount,
                                f"Слишком большая скидка: {discount/total*100:.0f}%")
            
            valid_data.append(row)
        
        return valid_data


class OrderItemValidator(CustomValidator):
    """Валидация товаров в заказе."""
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = []
        
        for idx, row in enumerate(data):
            items = row.get('items', [])
            
            if not items:
                self.add_error(idx, 'items', items,
                              "Заказ должен содержать хотя бы один товар")
                continue
            
            total_calculated = 0
            
            for item_idx, item in enumerate(items):
                quantity = item.get('quantity', 0)
                price = item.get('price', 0)
                
                if quantity <= 0:
                    self.add_error(idx, f'items[{item_idx}].quantity', quantity,
                                  "Количество товара должно быть положительным")
                
                if price < 0:
                    self.add_error(idx, f'items[{item_idx}].price', price,
                                  "Цена товара не может быть отрицательной")
                
                total_calculated += quantity * price
            
            # Проверка соответствия общей суммы
            expected_total = row.get('total', 0)
            if abs(total_calculated - expected_total) > 0.01:
                self.add_warning(idx, 'total', expected_total,
                                f"Расчётная сумма {total_calculated} не совпадает с заявленной {expected_total}")
                row['_calculated_total'] = total_calculated
            
            valid_data.append(row)
        
        return valid_data


# ============================================================
# ПРИМЕР ИСПОЛЬЗОВАНИЯ
# ============================================================

def validate_orders():
    """Пример валидации заказов."""
    
    # Тестовые данные
    test_orders = [
        {
            "order_id": 1,
            "order_date": datetime(2024, 12, 31),  # будущая дата
            "delivery_date": datetime(2024, 12, 25),
            "payment_method": "card",
            "card_number": "1234",
            "total": 1000,
            "discount": 1500,  # скидка больше суммы
            "items": [
                {"name": "Товар 1", "quantity": 1, "price": 1000}
            ]
        },
        {
            "order_id": 2,
            "order_date": datetime(2024, 1, 15),
            "delivery_date": datetime(2024, 1, 20),
            "payment_method": "cash",
            "total": 500,
            "discount": 50,
            "items": []
        },
        {
            "order_id": 3,
            "order_date": datetime(2024, 2, 1),
            "delivery_date": datetime(2024, 2, 10),
            "payment_method": "card",
            "card_number": "1234567890123456",
            "card_expiry": "12/25",
            "total": 3000,
            "discount": 500,
            "items": [
                {"name": "Товар A", "quantity": 2, "price": 1000},
                {"name": "Товар B", "quantity": 1, "price": 1500}
            ]
        }
    ]
    
    # Создание иерархии валидаторов
    validators = [
        OrderDateValidator(),
        PaymentValidator(),
        DiscountValidator(),
        OrderItemValidator()
    ]
    
    # Запуск валидации
    valid_data = test_orders
    
    for validator in validators:
        valid_data = validator.validate(valid_data)
        
        for error in validator.get_errors():
            print(f"❌ Ошибка: {error.message}")
        
        for warning in validator.get_warnings():
            print(f"⚠️ Предупреждение: {warning.message}")
    
    print(f"\n✅ Успешно валидировано: {len(valid_data)} из {len(test_orders)} заказов")
    
    return valid_data


if __name__ == "__main__":
    validate_orders()
5.2. Ожидаемый вывод
text
❌ Ошибка: Дата заказа не может быть в будущем
⚠️ Предупреждение: Нестандартная длина номера карты
❌ Ошибка: Скидка 1500 не может превышать сумму заказа 1000
❌ Ошибка: Заказ должен содержать хотя бы один товар

✅ Успешно валидировано: 1 из 3 заказов
6. Контрольные вопросы
Когда нужно создавать кастомные валидаторы вместо использования стандартных?

Как реализовать валидацию зависимости между несколькими полями?

В чём разница между ошибкой и предупреждением в валидации?

Как организовать иерархию валидаторов с разными уровнями критичности?

Что такое корректирующий валидатор и когда его использовать?

Как интегрировать валидацию в существующий ETL-процесс?

Как генерировать отчёты о результатах валидации?

Как обрабатывать условную валидацию (валидация зависит от значения другого поля)?

Как тестировать кастомные валидаторы?

Как обеспечить производительность при валидации больших объёмов данных?

7. Практическое задание
Задание 1 (базовое)
Создайте валидатор для проверки формата телефонных номеров (российские номера) с нормализацией.

Задание 2 (среднее)
Реализуйте валидатор для проверки диапазона дат с учётом бизнес-правил (например, нельзя бронировать отель более чем на 30 дней).

Задание 3 (сложное)
Разработайте полноценную систему валидации для интернет-магазина:

Проверка наличия товара

Валидация суммы заказа

Проверка корректности промокодов

Ограничение на количество товаров

Валидация адреса доставки

8. Шпаргалка
python
# === КАСТОМНЫЙ ВАЛИДАТОР ===
class MyValidator(CustomValidator):
    def validate(self, data):
        valid_data = []
        for idx, row in enumerate(data):
            if self._check(row):
                self.add_error(idx, "field", value, "error")
            else:
                valid_data.append(row)
        return valid_data

# === ВАЛИДАЦИЯ МЕЖДУ ПОЛЯМИ ===
class CrossFieldValidator(CustomValidator):
    def validate(self, data):
        for idx, row in enumerate(data):
            if row['start'] >= row['end']:
                self.add_error(idx, "start,end", row, "start < end required")

# === УСЛОВНАЯ ВАЛИДАЦИЯ ===
class ConditionalValidator(CustomValidator):
    def __init__(self, field, value, validator):
        self.field = field
        self.value = value
        self.validator = validator

# === ИЕРАРХИЧЕСКАЯ ВАЛИДАЦИЯ ===
validator = HierarchicalValidator({
    ValidationLevel.SCHEMA: [schema_validator],
    ValidationLevel.TYPE: [type_validator],
    ValidationLevel.BUSINESS: [business_validator]
})
# === ИСПРАВЛЯЮЩИЙ ВАЛИДАТОР ===
class FixingValidator(CorrectiveValidator):
    def _fix_row(self, row, idx):
        if condition:
            row['field'] = fixed_value
            return row, True
        return row, False
Итог лекции
Вы сегодня:

✅ Изучили создание кастомных валидаторов для специфических правил

✅ Реализовали валидацию зависимостей между полями

✅ Освоили иерархическую валидацию с разными уровнями

✅ Интегрировали валидацию в ETL-процесс

✅ Создали корректирующие валидаторы для нормализации данных

Теперь вы можете создавать гибкие и мощные системы валидации для любых бизнес-правил!
