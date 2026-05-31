# Тема 2.29. Комплексная обработка данных: ETL + валидация. Архитектура ETL-модуля. Разделение на слои

**Цель лекции:**  
Изучить комплексный подход к обработке данных, объединяющий ETL-процессы и валидацию. Освоить архитектуру ETL-модуля с разделением на слои (Extract, Transform, Load, Validation). Научиться проектировать масштабируемые и поддерживаемые конвейеры обработки данных.

> Главная мысль: **ETL без валидации — это риск. Валидация без ETL — это хаос. Только вместе они создают надёжную систему обработки данных.**

---

## Содержание

1. [Введение в комплексную обработку данных](#1-введение-в-комплексную-обработку-данных)
2. [Архитектура ETL-модуля](#2-архитектура-etl-модуля)
3. [Разделение на слои](#3-разделение-на-слои)
4. [Валидация данных в ETL-процессе](#4-валидация-данных-в-etl-процессе)
5. [Интеграция компонентов](#5-интеграция-компонентов)
6. [Практические примеры](#6-практические-примеры)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Введение в комплексную обработку данных

### 1.1. Проблематика обработки данных
┌─────────────────────────────────────────────────────────────────────────────┐
│ ПРОБЛЕМЫ НЕСТРУКТУРИРОВАННОЙ ОБРАБОТКИ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ┌─────────────┐ │
│ │ CSV файлы │ ──┐ │
│ └─────────────┘ │ │
│ ┌─────────────┐ │ ┌─────────────────────────────────────┐ │
│ │ JSON API │ ──┼───►│ БЕССИСТЕМНАЯ ОБРАБОТКА │ │
│ └─────────────┘ │ │ ❌ Нет стандартов │ │
│ ┌─────────────┐ │ │ ❌ Дублирование кода │ │
│ │ База данных│ ──┘ │ ❌ Трудно тестировать │ │
│ └─────────────┘ │ ❌ Невозможно переиспользовать │ │
│ └─────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Комплексный подход

| Проблема | Решение |
|----------|---------|
| Разнородные источники | Единый интерфейс Extract |
| Грязные данные | Слой валидации |
| Дублирование логики | Переиспользуемые трансформации |
| Отсутствие отказоустойчивости | Логирование и обработка ошибок |
| Мониторинг | Метрики и алерты |

### 1.3. Принципы комплексной обработки

```python
# Плохо: всё в одном файле
def process_data():
    # Чтение CSV
    data = []
    with open('data.csv') as f:
        reader = csv.DictReader(f)
        for row in reader:
            # Валидация размазана по коду
            if not row['name']:
                continue
            if int(row['age']) < 0:
                continue
            # Трансформация
            row['age'] = int(row['age'])
            data.append(row)
    # Запись в JSON
    with open('output.json', 'w') as f:
        json.dump(data, f)

# Хорошо: разделение на слои
def extract():
    pass

def validate():
    pass

def transform():
    pass

def load():
    pass

def run_etl():
    data = extract()
    valid_data = validate(data)
    transformed = transform(valid_data)
    load(transformed)
2. Архитектура ETL-модуля
2.1. Общая архитектура
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                         АРХИТЕКТУРА ETL-МОДУЛЯ                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         КОМПОНЕНТЫ ETL                               │   │
│  │                                                                     │   │
│  │  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐      │   │
│  │  │ Extract  │───►│ Validate │───►│Transform │───►│   Load   │      │   │
│  │  │ ┌──────┐ │    │ ┌──────┐ │    │ ┌──────┐ │    │ ┌──────┐ │      │   │
│  │  │ │CSV   │ │    │ │Схема │ │    │ │Очистка│ │    │ │JSON  │ │      │   │
│  │  │ │JSON  │ │    │ │Типы  │ │    │ │Норма-│ │    │ │БД    │ │      │   │
│  │  │ │API   │ │    │ │Бизнес│ │    │ │лизация│ │    │ │API   │ │      │   │
│  │  │ │БД    │ │    │ └──────┘ │    │ └──────┘ │    │ └──────┘ │      │   │
│  │  │ └──────┘ │    └──────────┘    └──────────┘    └──────────┘      │   │
│  │  └──────────┘                                                      │   │
│  │                                                                     │   │
│  │  ┌─────────────────────────────────────────────────────────────┐   │   │
│  │  │                    СКВОЗНЫЕ КОМПОНЕНТЫ                       │   │   │
│  │  │  ┌──────────┐    ┌──────────┐    ┌──────────┐              │   │   │
│  │  │  │Логирова- │    │Метрики/  │    │Обработка │              │   │   │
│  │  │  │ние       │    │Статистика│    │ошибок    │              │   │   │
│  │  │  └──────────┘    └──────────┘    └──────────┘              │   │   │
│  │  └─────────────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
2.2. Ключевые компоненты
Компонент	Ответственность	Примеры
Extractor	Извлечение данных из источников	CSVReader, APIClient, DatabaseConnector
Validator	Проверка качества данных	SchemaValidator, TypeValidator, BusinessValidator
Transformer	Преобразование данных	Cleaner, Normalizer, Aggregator, Enricher
Loader	Загрузка в целевые системы	JSONWriter, DatabaseInserter, APIClient
Pipeline	Оркестрация процесса	SequentialPipeline, ParallelPipeline
Logger	Логирование хода выполнения	FileLogger, DatabaseLogger
Metrics	Сбор статистики	Counter, Timer, Gauge
3. Разделение на слои
3.1. Слои архитектуры
python
"""
Структура ETL-модуля:

etl_module/
├── extract/
│   ├── base.py          # Базовый класс Extractor
│   ├── csv_extractor.py # CSV извлечение
│   ├── json_extractor.py# JSON извлечение
│   └── api_extractor.py # API извлечение
├── validate/
│   ├── base.py          # Базовый класс Validator
│   ├── schema.py        # Валидация схемы
│   ├── type_validator.py# Валидация типов
│   └── business.py      # Бизнес-правила
├── transform/
│   ├── base.py          # Базовый класс Transformer
│   ├── clean.py         # Очистка данных
│   ├── normalize.py     # Нормализация
│   ├── aggregate.py     # Агрегация
│   └── enrich.py        # Обогащение
├── load/
│   ├── base.py          # Базовый класс Loader
│   ├── json_loader.py   # JSON загрузка
│   ├── db_loader.py     # Загрузка в БД
│   └── api_loader.py    # API загрузка
├── pipeline/
│   ├── base.py          # Базовый класс Pipeline
│   ├── sequential.py    # Последовательный пайплайн
│   └── parallel.py      # Параллельный пайплайн
├── core/
│   ├── models.py        # Модели данных
│   ├── exceptions.py    # Исключения
│   └── config.py        # Конфигурация
└── utils/
    ├── logger.py        # Логирование
    ├── metrics.py       # Метрики
    └── errors.py        # Обработка ошибок
"""
3.2. Базовые классы
python
# extract/base.py
from abc import ABC, abstractmethod
from typing import Iterator, Dict, Any, Optional
from dataclasses import dataclass
from datetime import datetime


@dataclass
class ExtractResult:
    """Результат извлечения."""
    data: Iterator[Dict[str, Any]]
    total_rows: int
    source: str
    timestamp: datetime


class Extractor(ABC):
    """Базовый класс для всех экстракторов."""
    
    def __init__(self, source: str, config: Optional[Dict] = None):
        self.source = source
        self.config = config or {}
    
    @abstractmethod
    def extract(self) -> ExtractResult:
        """Извлечение данных из источника."""
        pass
    
    @abstractmethod
    def validate_source(self) -> bool:
        """Проверка доступности источника."""
        pass
    
    def get_metadata(self) -> Dict:
        """Метаданные экстрактора."""
        return {
            "source": self.source,
            "type": self.__class__.__name__,
            "config": self.config
        }
python
# validate/base.py
from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional
from dataclasses import dataclass


@dataclass
class ValidationError:
    """Ошибка валидации."""
    row_index: int
    field: str
    value: Any
    message: str
    error_type: str


class Validator(ABC):
    """Базовый класс для валидаторов."""
    
    def __init__(self, config: Optional[Dict] = None):
        self.config = config or {}
        self.errors: List[ValidationError] = []
    
    @abstractmethod
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        """
        Валидация данных.
        
        Returns:
            Валидные данные
        """
        pass
    
    def get_errors(self) -> List[ValidationError]:
        """Получение списка ошибок."""
        return self.errors
    
    def has_errors(self) -> bool:
        """Есть ли ошибки валидации."""
        return len(self.errors) > 0
python
# transform/base.py
from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional


class Transformer(ABC):
    """Базовый класс для трансформеров."""
    
    def __init__(self, config: Optional[Dict] = None):
        self.config = config or {}
    
    @abstractmethod
    def transform(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        """
        Преобразование данных.
        
        Returns:
            Трансформированные данные
        """
        pass
    
    def can_handle(self, data: List[Dict[str, Any]]) -> bool:
        """Может ли трансформер обработать данные."""
        return True
python
# load/base.py
from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional
from dataclasses import dataclass


@dataclass
class LoadResult:
    """Результат загрузки."""
    destination: str
    rows_loaded: int
    timestamp: datetime
    success: bool
    message: Optional[str] = None


class Loader(ABC):
    """Базовый класс для загрузчиков."""
    
    def __init__(self, destination: str, config: Optional[Dict] = None):
        self.destination = destination
        self.config = config or {}
    
    @abstractmethod
    def load(self, data: List[Dict[str, Any]]) -> LoadResult:
        """Загрузка данных в целевое хранилище."""
        pass
    
    @abstractmethod
    def validate_destination(self) -> bool:
        """Проверка доступности целевого хранилища."""
        pass
4. Валидация данных в ETL-процессе
4.1. Типы валидации
Тип валидации	Описание	Инструменты
Схема	Проверка наличия полей	Pydantic, Marshmallow
Типы данных	Проверка соответствия типов	isinstance, type hints
Формат	Проверка формата (email, дата)	regex, datetime
Диапазон	Проверка значений в границах	min, max, gt, lt
Уникальность	Проверка дубликатов	set, groupby
Бизнес-правила	Специфичные правила	Пользовательские функции
4.2. Композитный валидатор
python
# validate/composite.py
from typing import List, Dict, Any, Optional
from .base import Validator, ValidationError


class CompositeValidator(Validator):
    """Композитный валидатор, объединяющий несколько валидаторов."""
    
    def __init__(self, validators: List[Validator], config: Optional[Dict] = None):
        super().__init__(config)
        self.validators = validators
        self.stop_on_first_error = config.get("stop_on_first_error", False)
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = data
        
        for validator in self.validators:
            valid_data = validator.validate(valid_data)
            self.errors.extend(validator.get_errors())
            
            if self.stop_on_first_error and validator.has_errors():
                break
        
        return valid_data


class SchemaValidator(Validator):
    """Валидатор схемы данных."""
    
    def __init__(self, required_fields: List[str], config: Optional[Dict] = None):
        super().__init__(config)
        self.required_fields = required_fields
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = []
        
        for idx, row in enumerate(data):
            missing_fields = [f for f in self.required_fields if f not in row]
            
            if missing_fields:
                self.errors.append(ValidationError(
                    row_index=idx,
                    field=",".join(missing_fields),
                    value=row,
                    message=f"Отсутствуют обязательные поля: {missing_fields}",
                    error_type="schema"
                ))
                continue
            
            valid_data.append(row)
        
        return valid_data


class TypeValidator(Validator):
    """Валидатор типов данных."""
    
    def __init__(self, field_types: Dict[str, type], config: Optional[Dict] = None):
        super().__init__(config)
        self.field_types = field_types
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = []
        
        for idx, row in enumerate(data):
            row_valid = True
            
            for field, expected_type in self.field_types.items():
                if field in row:
                    value = row[field]
                    
                    if not isinstance(value, expected_type):
                        self.errors.append(ValidationError(
                            row_index=idx,
                            field=field,
                            value=value,
                            message=f"Ожидался тип {expected_type.__name__}, получен {type(value).__name__}",
                            error_type="type"
                        ))
                        row_valid = False
            
            if row_valid:
                valid_data.append(row)
        
        return valid_data


class RangeValidator(Validator):
    """Валидатор диапазонов значений."""
    
    def __init__(self, field_ranges: Dict[str, Dict], config: Optional[Dict] = None):
        super().__init__(config)
        self.field_ranges = field_ranges
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = []
        
        for idx, row in enumerate(data):
            row_valid = True
            
            for field, ranges in self.field_ranges.items():
                if field in row:
                    value = row[field]
                    
                    if "min" in ranges and value < ranges["min"]:
                        self.errors.append(ValidationError(
                            row_index=idx,
                            field=field,
                            value=value,
                            message=f"Значение {value} меньше минимального {ranges['min']}",
                            error_type="range"
                        ))
                        row_valid = False
                    
                    if "max" in ranges and value > ranges["max"]:
                        self.errors.append(ValidationError(
                            row_index=idx,
                            field=field,
                            value=value,
                            message=f"Значение {value} больше максимального {ranges['max']}",
                            error_type="range"
                        ))
                        row_valid = False
            
            if row_valid:
                valid_data.append(row)
        
        return valid_data


class UniqueValidator(Validator):
    """Валидатор уникальности полей."""
    
    def __init__(self, unique_fields: List[str], config: Optional[Dict] = None):
        super().__init__(config)
        self.unique_fields = unique_fields
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        seen = {}
        valid_data = []
        
        for idx, row in enumerate(data):
            is_duplicate = False
            
            for field in self.unique_fields:
                if field in row:
                    key = f"{field}:{row[field]}"
                    
                    if key in seen:
                        self.errors.append(ValidationError(
                            row_index=idx,
                            field=field,
                            value=row[field],
                            message=f"Дубликат значения '{row[field]}' в поле '{field}'",
                            error_type="unique"
                        ))
                        is_duplicate = True
                    else:
                        seen[key] = idx
            
            if not is_duplicate:
                valid_data.append(row)
        
        return valid_data
4.3. Использование Pydantic для валидации
python
# validate/pydantic_validator.py
from pydantic import BaseModel, Field, validator, EmailStr
from typing import List, Dict, Any, Optional
from datetime import datetime
from .base import Validator, ValidationError


class UserSchema(BaseModel):
    """Pydantic схема для валидации пользователей."""
    
    id: int = Field(..., gt=0)
    name: str = Field(..., min_length=2, max_length=100)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)
    city: Optional[str] = None
    created_at: datetime
    
    @validator('name')
    def name_not_empty(cls, v):
        if not v.strip():
            raise ValueError('Имя не может быть пустым')
        return v.strip().title()


class ProductSchema(BaseModel):
    """Pydantic схема для валидации товаров."""
    
    id: int = Field(..., gt=0)
    name: str = Field(..., min_length=1, max_length=200)
    price: float = Field(..., gt=0)
    stock: int = Field(..., ge=0)
    category_id: int = Field(..., gt=0)


class PydanticValidator(Validator):
    """Валидатор с использованием Pydantic."""
    
    def __init__(self, schema_class: BaseModel, config: Optional[Dict] = None):
        super().__init__(config)
        self.schema_class = schema_class
    
    def validate(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        valid_data = []
        
        for idx, row in enumerate(data):
            try:
                validated = self.schema_class(**row)
                valid_data.append(validated.dict())
            except Exception as e:
                self.errors.append(ValidationError(
                    row_index=idx,
                    field="all",
                    value=row,
                    message=str(e),
                    error_type="pydantic"
                ))
        
        return valid_data
5. Интеграция компонентов
5.1. Класс Pipeline
python
# pipeline/sequential.py
from typing import List, Dict, Any, Optional, Callable
from datetime import datetime
from ..extract.base import Extractor, ExtractResult
from ..validate.base import Validator
from ..transform.base import Transformer
from ..load.base import Loader, LoadResult
from ..utils.logger import ETLlogger
from ..utils.metrics import MetricsCollector


class ETLPipeline:
    """Основной класс ETL-пайплайна."""
    
    def __init__(
        self,
        name: str,
        extractor: Extractor,
        validators: List[Validator],
        transformers: List[Transformer],
        loader: Loader,
        config: Optional[Dict] = None
    ):
        self.name = name
        self.extractor = extractor
        self.validators = validators
        self.transformers = transformers
        self.loader = loader
        self.config = config or {}
        
        self.logger = ETLlogger(f"pipeline.{name}")
        self.metrics = MetricsCollector(f"pipeline.{name}")
        
        self.extract_result: Optional[ExtractResult] = None
        self.load_result: Optional[LoadResult] = None
        self.validation_errors_count = 0
    
    def run(self) -> bool:
        """Запуск ETL-пайплайна."""
        start_time = datetime.now()
        self.logger.info(f"Запуск ETL-пайплайна '{self.name}'")
        
        try:
            # 1. EXTRACT
            self.logger.info("Этап 1: Извлечение данных")
            if not self.extractor.validate_source():
                self.logger.error("Источник данных недоступен")
                return False
            
            self.extract_result = self.extractor.extract()
            self.logger.info(f"Извлечено {self.extract_result.total_rows} записей")
            self.metrics.gauge("extracted_rows", self.extract_result.total_rows)
            
            # 2. VALIDATE
            self.logger.info("Этап 2: Валидация данных")
            data = list(self.extract_result.data)
            
            for validator in self.validators:
                data = validator.validate(data)
                self.validation_errors_count += len(validator.get_errors())
                
                if validator.has_errors():
                    self.logger.warning(f"Найдено {len(validator.get_errors())} ошибок валидации")
                    
                    if self.config.get("fail_on_validation_error", False):
                        self.logger.error("Остановка из-за ошибок валидации")
                        return False
            
            self.logger.info(f"После валидации осталось {len(data)} записей")
            self.metrics.gauge("validated_rows", len(data))
            
            # 3. TRANSFORM
            self.logger.info("Этап 3: Трансформация данных")
            for transformer in self.transformers:
                data = transformer.transform(data)
                self.logger.debug(f"Трансформация {transformer.__class__.__name__} выполнена")
            
            self.logger.info(f"После трансформации: {len(data)} записей")
            
            # 4. LOAD
            self.logger.info("Этап 4: Загрузка данных")
            if not self.loader.validate_destination():
                self.logger.error("Целевое хранилище недоступно")
                return False
            
            self.load_result = self.loader.load(data)
            
            if self.load_result.success:
                self.logger.info(f"Загружено {self.load_result.rows_loaded} записей")
            else:
                self.logger.error(f"Ошибка загрузки: {self.load_result.message}")
                return False
            
            # 5. ИТОГИ
            end_time = datetime.now()
            duration = (end_time - start_time).total_seconds()
            
            self.logger.info(f"Пайплайн выполнен за {duration:.2f} секунд")
            self.metrics.timer("pipeline_duration", duration)
            self.metrics.gauge("validation_errors", self.validation_errors_count)
            self.metrics.gauge("loaded_rows", self.load_result.rows_loaded)
            
            return True
            
        except Exception as e:
            self.logger.error(f"Критическая ошибка: {e}", exc_info=True)
            self.metrics.counter("pipeline_errors", 1)
            return False
    
    def get_report(self) -> Dict:
        """Получение отчёта о выполнении."""
        return {
            "name": self.name,
            "extracted_rows": self.extract_result.total_rows if self.extract_result else 0,
            "validation_errors": self.validation_errors_count,
            "loaded_rows": self.load_result.rows_loaded if self.load_result else 0,
            "success": self.load_result.success if self.load_result else False,
            "metrics": self.metrics.get_all()
        }
5.2. Логирование и метрики
python
# utils/logger.py
import logging
import sys
from pathlib import Path


class ETLlogger:
    """Логгер для ETL-процессов."""
    
    def __init__(self, name: str, log_dir: str = "logs"):
        self.logger = logging.getLogger(name)
        self.logger.setLevel(logging.DEBUG)
        
        # Консольный обработчик
        console_handler = logging.StreamHandler(sys.stdout)
        console_handler.setLevel(logging.INFO)
        console_format = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
        console_handler.setFormatter(console_format)
        self.logger.addHandler(console_handler)
        
        # Файловый обработчик
        Path(log_dir).mkdir(exist_ok=True)
        file_handler = logging.FileHandler(f"{log_dir}/{name}.log", encoding='utf-8')
        file_handler.setLevel(logging.DEBUG)
        file_format = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
        file_handler.setFormatter(file_format)
        self.logger.addHandler(file_handler)
    
    def debug(self, message: str):
        self.logger.debug(message)
    
    def info(self, message: str):
        self.logger.info(message)
    
    def warning(self, message: str):
        self.logger.warning(message)
    
    def error(self, message: str, exc_info: bool = False):
        self.logger.error(message, exc_info=exc_info)
python
# utils/metrics.py
from collections import defaultdict
from datetime import datetime
from typing import Dict, Any
import json


class MetricsCollector:
    """Сборщик метрик для ETL-процессов."""
    
    def __init__(self, name: str):
        self.name = name
        self.metrics = {
            "gauges": {},
            "counters": defaultdict(int),
            "timers": []
        }
    
    def gauge(self, key: str, value: Any):
        """Установка текущего значения метрики."""
        self.metrics["gauges"][key] = value
    
    def counter(self, key: str, value: int = 1):
        """Инкремент счётчика."""
        self.metrics["counters"][key] += value
    
    def timer(self, key: str, duration: float):
        """Запись времени выполнения."""
        self.metrics["timers"].append({
            "key": key,
            "duration": duration,
            "timestamp": datetime.now().isoformat()
        })
    
    def get_all(self) -> Dict:
        """Получение всех метрик."""
        return {
            "gauges": self.metrics["gauges"],
            "counters": dict(self.metrics["counters"]),
            "timers": self.metrics["timers"][-100:]  # последние 100
        }
    
    def get_summary(self) -> Dict:
        """Краткая сводка метрик."""
        timer_durations = [t["duration"] for t in self.metrics["timers"]]
        
        return {
            "name": self.name,
            "gauges": self.metrics["gauges"],
            "counters": dict(self.metrics["counters"]),
            "avg_timer_duration": sum(timer_durations) / len(timer_durations) if timer_durations else 0
        }
6. Практические примеры
6.1. Полный ETL-пайплайн для пользовательских данных
python
# example_user_etl.py
"""
Полный пример ETL-пайплайна для обработки пользовательских данных.
"""

import csv
import json
from datetime import datetime
from typing import List, Dict, Any, Iterator
from pathlib import Path

# Импорт компонентов
from extract.base import Extractor, ExtractResult
from validate.base import Validator
from validate.composite import (
    CompositeValidator, SchemaValidator, TypeValidator, 
    RangeValidator, UniqueValidator
)
from validate.pydantic_validator import PydanticValidator, UserSchema
from transform.base import Transformer
from load.base import Loader, LoadResult
from pipeline.sequential import ETLPipeline


# ============================================================
# КОМПОНЕНТ EXTRACT
# ============================================================

class CSVExtractor(Extractor):
    """Экстрактор данных из CSV файла."""
    
    def extract(self) -> ExtractResult:
        data = []
        
        with open(self.source, 'r', encoding='utf-8') as f:
            reader = csv.DictReader(f)
            for row in reader:
                # Преобразование строковых значений
                row['id'] = int(row.get('id', 0))
                row['age'] = int(row.get('age', 0))
                data.append(row)
        
        return ExtractResult(
            data=iter(data),
            total_rows=len(data),
            source=self.source,
            timestamp=datetime.now()
        )
    
    def validate_source(self) -> bool:
        return Path(self.source).exists()


# ============================================================
# КОМПОНЕНТ TRANSFORM
# ============================================================

class DateNormalizer(Transformer):
    """Нормализатор дат."""
    
    def transform(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        for row in data:
            if 'created_at' in row and row['created_at']:
                # Попытка преобразовать строку в дату
                try:
                    if isinstance(row['created_at'], str):
                        row['created_at'] = datetime.fromisoformat(row['created_at'])
                except ValueError:
                    row['created_at'] = datetime.now()
            else:
                row['created_at'] = datetime.now()
        
        return data


class NameNormalizer(Transformer):
    """Нормализатор имён."""
    
    def transform(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        for row in data:
            if 'name' in row:
                row['name'] = row['name'].strip().title()
        
        return data


class AgeGroupEnricher(Transformer):
    """Обогатитель данных — добавление возрастной группы."""
    
    def transform(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        for row in data:
            age = row.get('age', 0)
            
            if age < 18:
                row['age_group'] = 'child'
            elif age < 35:
                row['age_group'] = 'young'
            elif age < 60:
                row['age_group'] = 'adult'
            else:
                row['age_group'] = 'senior'
        
        return data


# ============================================================
# КОМПОНЕНТ LOAD
# ============================================================

class JSONLoader(Loader):
    """Загрузчик данных в JSON файл."""
    
    def load(self, data: List[Dict[str, Any]]) -> LoadResult:
        try:
            # Сериализация datetime объектов
            def serialize(obj):
                if isinstance(obj, datetime):
                    return obj.isoformat()
                return obj
            
            with open(self.destination, 'w', encoding='utf-8') as f:
                json.dump(data, f, indent=2, ensure_ascii=False, default=serialize)
            
            return LoadResult(
                destination=self.destination,
                rows_loaded=len(data),
                timestamp=datetime.now(),
                success=True
            )
        except Exception as e:
            return LoadResult(
                destination=self.destination,
                rows_loaded=0,
                timestamp=datetime.now(),
                success=False,
                message=str(e)
            )
    
    def validate_destination(self) -> bool:
        dest_path = Path(self.destination)
        return dest_path.parent.exists()


# ============================================================
# ЗАПУСК ПАЙПЛАЙНА
# ============================================================

def run_user_etl():
    """Запуск ETL-пайплайна для пользовательских данных."""
    
    # Создание CSV файла с тестовыми данными
    test_csv = Path("users_test.csv")
    with open(test_csv, 'w', encoding='utf-8') as f:
        f.write("id,name,email,age,city,created_at\n")
        f.write("1,иван петров,ivan@example.com,25,Москва,2024-01-15T10:30:00\n")
        f.write("2,мария сидорова,maria@example.com,30,СПб,2024-01-16T11:00:00\n")
        f.write("3,петр,invalid-email,28,Казань,2024-01-17T09:00:00\n")
        f.write("4,анна,anna@example.com,-5,Москва,\n")
        f.write("5,дмитрий,dmitry@example.com,35,Новосибирск,2024-01-18T14:30:00\n")
    
    ## Компоненты
    extractor = CSVExtractor(source=str(test_csv))
    
    validators = [
        SchemaValidator(required_fields=['id', 'name', 'email', 'age']),
        TypeValidator(field_types={'id': int, 'age': int}),
        RangeValidator(field_ranges={'age': {'min': 0, 'max': 150}}),
        UniqueValidator(unique_fields=['email']),
        PydanticValidator(schema_class=UserSchema)
    ]
    
    transformers = [
        NameNormalizer(),
        DateNormalizer(),
        AgeGroupEnricher()
    ]
    
    loader = JSONLoader(destination="output/users_output.json")
    
    # Пайплайн
    pipeline = ETLPipeline(
        name="user_etl",
        extractor=extractor,
        validators=validators,
        transformers=transformers,
        loader=loader,
        config={"fail_on_validation_error": False}
    )
    
    # Запуск
    success = pipeline.run()
    
    # Отчёт
    report = pipeline.get_report()
    print("\n" + "=" * 60)
    print("ОТЧЁТ О ВЫПОЛНЕНИИ ETL-ПАЙПЛАЙНА")
    print("=" * 60)
    print(f"Пайплайн: {report['name']}")
    print(f"Извлечено записей: {report['extracted_rows']}")
    print(f"Ошибок валидации: {report['validation_errors']}")
    print(f"Загружено записей: {report['loaded_rows']}")
    print(f"Успех: {report['success']}")
    
    return success


if __name__ == "__main__":
    run_user_etl()
7. Контрольные вопросы
Какие основные компоненты входят в архитектуру ETL-модуля?

Зачем нужно разделение на слои в ETL-процессе?

Какие типы валидации данных вы знаете?

В чём разница между Extractor и Loader?

Как организовать обработку ошибок в ETL-пайплайне?

Что такое композитный валидатор и когда он используется?

Какие метрики важно собирать в ETL-процессе?

Как интегрировать Pydantic в ETL для валидации?

Почему важно разделять трансформации на отдельные компоненты?

Как тестировать отдельные компоненты ETL?

8. Практическое задание
Задание 1 (базовое)
Создайте ETL-пайплайн для обработки CSV файла с данными о товарах. Реализуйте валидацию схемы и типов.

Задание 2 (среднее)
Расширьте пайплайн: добавьте трансформации (нормализация цен, расчёт скидки) и загрузку в JSON.

Задание 3 (сложное)
Создайте полноценный ETL-модуль для интернет-магазина:

Extract: CSV с товарами, JSON с заказами

Validate: Pydantic схемы, бизнес-правила

Transform: расчёт выручки, группировка по категориям

Load: загрузка в SQLite и JSON отчёт

9. Шпаргалка
python
# === БАЗОВАЯ СТРУКТУРА ETL ===
class Extractor(ABC):
    @abstractmethod
    def extract(self): pass

class Validator(ABC):
    @abstractmethod
    def validate(self, data): pass

class Transformer(ABC):
    @abstractmethod
    def transform(self, data): pass

class Loader(ABC):
    @abstractmethod
    def load(self, data): pass

class Pipeline:
    def run(self):
        data = self.extractor.extract()
        data = self.validator.validate(data)
        data = self.transformer.transform(data)
        self.loader.load(data)

# === КОМПОЗИТНЫЙ ВАЛИДАТОР ===
validator = CompositeValidator([
    SchemaValidator(required_fields),
    TypeValidator(field_types),
    RangeValidator(field_ranges)
])

# === PYDANTIC ВАЛИДАЦИЯ ===
class UserSchema(BaseModel):
    name: str = Field(..., min_length=2)
    age: int = Field(..., ge=0, le=150)

validator = PydanticValidator(UserSchema)

# === ЗАПУСК ПАЙПЛАЙНА ===
pipeline = ETLPipeline(name="my_etl", ...)
success = pipeline.run()
report = pipeline.get_report()
Итог лекции
Вы сегодня:

✅ Изучили архитектуру ETL-модуля с разделением на слои

✅ Освоили композитную валидацию данных

✅ Интегрировали Pydantic в ETL-процесс

✅ Создали переиспользуемые компоненты (Extractor, Validator, Transformer, Loader)

✅ Реализовали полноценный ETL-пайплайн с метриками и логированием

Теперь вы можете создавать масштабируемые и надёжные ETL-системы!
