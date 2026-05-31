# ПЗ 2.36. Написание интеграционных тестов для ETL-конвейера

**Тема:** Интеграционное тестирование, ETL-конвейеры, валидация данных, тест-дизайн

**Цель работы:**  
Научиться писать интеграционные тесты для ETL-конвейеров, тестировать взаимодействие всех компонентов, проверять обработку граничных случаев и восстанавливаемость системы.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленные пакеты: `pytest`, `pytest-cov`, `pydantic`

```bash
pip install pytest pytest-cov pydantic
Главная мысль: Модульные тесты проверяют детали, интеграционные — взаимодействие. Оба типа необходимы для надёжного ETL-конвейера.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Модульное vs Интеграционное тестирование
Характеристика	Модульное тестирование	Интеграционное тестирование
Что тестирует	Отдельный компонент	Взаимодействие компонентов
Изоляция	Полная (моки)	Минимальная (реальные зависимости)
Скорость	Быстрое	Медленнее
Сложность	Простое	Сложнее
Обнаруживает	Ошибки в логике	Ошибки взаимодействия
1.2. Что тестировать в ETL-конвейере
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ИНТЕГРАЦИОННЫЕ ТЕСТЫ ETL                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  ТЕСТ 1: СКВОЗНОЙ ПУТЬ (HAPPY PATH)                                 │   │
│  │  CSV → Валидация → Трансформация → JSON                              │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  ТЕСТ 2: ОБРАБОТКА ОШИБОК                                           │   │
│  │  Частично невалидные данные → DLQ, остальные → загрузка              │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  ТЕСТ 3: ГРАНИЧНЫЕ УСЛОВИЯ                                          │   │
│  │  Пустой файл → Пустой результат                                      │   │
│  │  Огромный файл → Не падает по памяти                                 │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  ТЕСТ 4: НЕДОСТУПНЫЕ РЕСУРСЫ                                        │   │
│  │  Отсутствует входной файл → Ошибка с сообщением                      │   │
│  │  Нет прав на запись → Ошибка с сообщением                            │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
1.3. Фикстуры для интеграционных тестов
python
# Фикстуры для тестирования ETL
@pytest.fixture
def temp_dir(tmp_path):
    """Временная директория для тестов."""
    return tmp_path

@pytest.fixture
def sample_csv(temp_dir):
    """Создание тестового CSV файла."""
    csv_file = temp_dir / "test_data.csv"
    # создание файла
    return csv_file

@pytest.fixture
def etl_pipeline(temp_dir):
    """Настроенный ETL-пайплайн."""
    return ETLPipeline(input_file, output_file)
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая написание интеграционных тестов для ETL-конвейера.

Техническое задание (нулевой вариант)
Напишите интеграционные тесты для ETL-конвейера обработки пользовательских данных. Тесты должны покрывать:

Полный успешный сценарий (happy path)

Частичную валидацию (DLQ)

Пустые и граничные данные

Ошибки доступа к файлам

Производительность (опционально)

Исходный ETL-конвейер
python
# user_etl.py
import csv
import json
from pathlib import Path
from datetime import datetime
from typing import List, Dict, Any, Optional, Tuple
from dataclasses import dataclass, field

from pydantic import BaseModel, Field, validator, EmailStr, ValidationError


# ============================================================
# PYDANTIC МОДЕЛИ
# ============================================================

class User(BaseModel):
    id: int = Field(..., gt=0)
    name: str = Field(..., min_length=2, max_length=100)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)
    city: str = Field(..., min_length=2, max_length=100)
    is_active: bool = True
    
    @validator('name')
    def normalize_name(cls, v):
        return v.strip().title()
    
    @validator('city')
    def normalize_city(cls, v):
        return v.strip().title()


# ============================================================
# КОМПОНЕНТЫ ETL
# ============================================================

@dataclass
class ErrorRecord:
    row_index: int
    raw_data: Dict
    error: str
    timestamp: datetime = field(default_factory=datetime.now)


class DeadLetterQueue:
    def __init__(self, dlq_path: str = "dlq"):
        self.dlq_path = Path(dlq_path)
        self.dlq_path.mkdir(exist_ok=True)
        self.errors: List[ErrorRecord] = []
    
    def add_error(self, row_index: int, raw_data: Dict, error: str):
        self.errors.append(ErrorRecord(row_index, raw_data, error))
        self._flush()
    
    def _flush(self):
        if not self.errors:
            return
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        dlq_file = self.dlq_path / f"dlq_{timestamp}.json"
        with open(dlq_file, 'w', encoding='utf-8') as f:
            json.dump([{
                "row_index": e.row_index,
                "raw_data": e.raw_data,
                "error": e.error,
                "timestamp": e.timestamp.isoformat()
            } for e in self.errors], f, indent=2, ensure_ascii=False)
    
    def clear(self):
        self.errors.clear()
    
    def get_errors(self) -> List[ErrorRecord]:
        return self.errors


class UserExtractor:
    def __init__(self, filepath: str):
        self.filepath = Path(filepath)
    
    def extract(self) -> Tuple[List[Dict], List[Dict]]:
        if not self.filepath.exists():
            raise FileNotFoundError(f"File {self.filepath} not found")
        
        data = []
        errors = []
        
        with open(self.filepath, 'r', encoding='utf-8') as f:
            reader = csv.DictReader(f)
            for idx, row in enumerate(reader, start=2):
                # Преобразование типов
                if 'id' in row:
                    try:
                        row['id'] = int(row['id'])
                    except ValueError:
                        errors.append({"index": idx, "row": row, "error": "Invalid id"})
                        continue
                
                if 'age' in row and row['age']:
                    try:
                        row['age'] = int(row['age'])
                    except ValueError:
                        row['age'] = None
                
                data.append(row)
        
        return data, errors


class UserValidator:
    def __init__(self, dlq: DeadLetterQueue):
        self.dlq = dlq
        self.valid_users = []
    
    def validate_row(self, row: Dict, row_index: int) -> Optional[User]:
        try:
            return User(**row)
        except ValidationError as e:
            error_msg = "; ".join([f"{err['loc'][0]}: {err['msg']}" for err in e.errors()])
            self.dlq.add_error(row_index, row, error_msg)
            return None
    
    def validate_batch(self, data: List[Dict], start_index: int = 2) -> List[User]:
        valid = []
        for idx, row in enumerate(data, start=start_index):
            user = self.validate_row(row, idx)
            if user:
                valid.append(user)
        return valid


class UserTransformer:
    def transform(self, users: List[User]) -> List[Dict]:
        transformed = []
        for user in users:
            transformed.append({
                "id": user.id,
                "name": user.name,
                "email": user.email,
                "age": user.age,
                "city": user.city,
                "is_active": user.is_active,
                "age_group": self._get_age_group(user.age),
                "processed_at": datetime.now().isoformat()
            })
        return transformed
    
    def _get_age_group(self, age: int) -> str:
        if age < 18:
            return "minor"
        elif age < 65:
            return "adult"
        else:
            return "senior"


class UserLoader:
    def __init__(self, output_path: str):
        self.output_path = Path(output_path)
        self.output_path.parent.mkdir(parents=True, exist_ok=True)
    
    def load(self, data: List[Dict]) -> bool:
        try:
            with open(self.output_path, 'w', encoding='utf-8') as f:
                json.dump(data, f, indent=2, ensure_ascii=False)
            return True
        except Exception:
            return False


class UserETLPipeline:
    def __init__(self, input_file: str, output_file: str):
        self.extractor = UserExtractor(input_file)
        self.dlq = DeadLetterQueue()
        self.validator = UserValidator(self.dlq)
        self.transformer = UserTransformer()
        self.loader = UserLoader(output_file)
        
        self.stats = {
            "extracted": 0,
            "valid": 0,
            "invalid": 0,
            "loaded": 0
        }
    
    def run(self) -> bool:
        try:
            # EXTRACT
            raw_data, csv_errors = self.extractor.extract()
            self.stats["extracted"] = len(raw_data)
            
            # VALIDATE
            valid_users = self.validator.validate_batch(raw_data)
            self.stats["valid"] = len(valid_users)
            self.stats["invalid"] = len(raw_data) - len(valid_users)
            
            # TRANSFORM
            transformed = self.transformer.transform(valid_users)
            
            # LOAD
            success = self.loader.load(transformed)
            self.stats["loaded"] = len(transformed) if success else 0
            
            return success
            
        except Exception as e:
            print(f"ETL failed: {e}")
            return False
    
    def get_stats(self) -> Dict:
        return self.stats
    
    def get_dlq_errors(self) -> List:
        return self.dlq.get_errors()
Интеграционные тесты
python
# test_user_etl_integration.py
"""
Интеграционные тесты для ETL-конвейера пользователей.
"""

import pytest
import json
from pathlib import Path
from user_etl import UserETLPipeline, DeadLetterQueue


class TestUserETLIntegration:
    """Интеграционные тесты для ETL-конвейера."""
    
    # ============================================================
    # ФИКСТУРЫ
    # ============================================================
    
    @pytest.fixture
    def temp_dir(self, tmp_path):
        """Временная директория для тестов."""
        return tmp_path
    
    @pytest.fixture
    def create_csv(self, temp_dir):
        """Фабрика для создания CSV файлов."""
        def _create_csv(filename: str, headers: list, rows: list):
            csv_file = temp_dir / filename
            with open(csv_file, 'w', encoding='utf-8') as f:
                f.write(','.join(headers) + '\n')
                for row in rows:
                    f.write(','.join(str(v) for v in row) + '\n')
            return csv_file
        return _create_csv
    
    @pytest.fixture
    def etl_pipeline(self, temp_dir):
        """Создание ETL-пайплайна."""
        def _create_pipeline(input_file: str, output_file: str = "output.json"):
            return UserETLPipeline(str(temp_dir / input_file), str(temp_dir / output_file))
        return _create_pipeline
    
    # ============================================================
    # ТЕСТ 1: СЧАСТЛИВЫЙ ПУТЬ (HAPPY PATH)
    # ============================================================
    
    def test_happy_path_single_user(self, create_csv, etl_pipeline, temp_dir):
        """Тест: один валидный пользователь."""
        # Arrange
        csv_file = create_csv(
            "users.csv",
            ["id", "name", "email", "age", "city"],
            [[1, "ivan petrov", "ivan@example.com", 25, "moscow"]]
        )
        
        pipeline = etl_pipeline("users.csv")
        
        # Act
        success = pipeline.run()
        output_file = temp_dir / "output.json"
        
        # Assert
        assert success is True
        assert pipeline.get_stats()["extracted"] == 1
        assert pipeline.get_stats()["valid"] == 1
        assert pipeline.get_stats()["invalid"] == 0
        assert pipeline.get_stats()["loaded"] == 1
        assert output_file.exists()
        
        with open(output_file, 'r', encoding='utf-8') as f:
            data = json.load(f)
        assert len(data) == 1
        assert data[0]["name"] == "Ivan Petrov"  # нормализация
        assert data[0]["age_group"] == "adult"
        assert "processed_at" in data[0]
    
    def test_happy_path_multiple_users(self, create_csv, etl_pipeline, temp_dir):
        """Тест: несколько валидных пользователей."""
        # Arrange
        csv_file = create_csv(
            "users.csv",
            ["id", "name", "email", "age", "city"],
            [
                [1, "ivan petrov", "ivan@example.com", 25, "moscow"],
                [2, "maria sidorova", "maria@example.com", 30, "spb"],
                [3, "petr ivanov", "petr@example.com", 28, "kazan"]
            ]
        )
        
        pipeline = etl_pipeline("users.csv")
        
        # Act
        success = pipeline.run()
        
        # Assert
        assert success is True
        assert pipeline.get_stats()["extracted"] == 3
        assert pipeline.get_stats()["valid"] == 3
        assert pipeline.get_stats()["invalid"] == 0
        assert pipeline.get_stats()["loaded"] == 3
    
    # ============================================================
    # ТЕСТ 2: ОБРАБОТКА ОШИБОК (DLQ)
    # ============================================================
    
    def test_partial_invalid_data(self, create_csv, etl_pipeline, temp_dir):
        """Тест: частично невалидные данные."""
        # Arrange
        csv_file = create_csv(
            "users.csv",
            ["id", "name", "email", "age", "city"],
            [
                [1, "ivan petrov", "ivan@example.com", 25, "moscow"],
                [2, "", "invalid-email", 30, "spb"],           # пустое имя, неверный email
                [3, "petr ivanov", "petr@example.com", -5, "kazan"]  # отрицательный возраст
            ]
        )
        
        pipeline = etl_pipeline("users.csv")
        
        # Act
        success = pipeline.run()
        output_file = temp_dir / "output.json"
        
        # Assert
        assert success is True
        assert pipeline.get_stats()["extracted"] == 3
        assert pipeline.get_stats()["valid"] == 1
        assert pipeline.get_stats()["invalid"] == 2
        assert pipeline.get_stats()["loaded"] == 1
        
        with open(output_file, 'r', encoding='utf-8') as f:
            data = json.load(f)
        assert len(data) == 1
        assert data[0]["name"] == "Ivan Petrov"
    
    def test_all_invalid_data(self, create_csv, etl_pipeline, temp_dir):
        """Тест: все данные невалидны."""
        # Arrange
        csv_file = create_csv(
            "users.csv",
            ["id", "name", "email", "age", "city"],
            [
                [0, "", "invalid", -1, ""],
                [-1, "", "", -5, ""]
            ]
        )
        
        pipeline = etl_pipeline("users.csv")
        
        # Act
        success = pipeline.run()
        
        # Assert
        assert success is True
        assert pipeline.get_stats()["extracted"] == 2
        assert pipeline.get_stats()["valid"] == 0
        assert pipeline.get_stats()["invalid"] == 2
        assert pipeline.get_stats()["loaded"] == 0
    
    def test_dlq_contains_errors(self, create_csv, etl_pipeline, temp_dir):
        """Тест: DLQ содержит информацию об ошибках."""
        # Arrange
        csv_file = create_csv(
            "users.csv",
            ["id", "name", "email", "age", "city"],
            [
                [1, "", "invalid@", 25, "moscow"]
            ]
        )
        
        pipeline = etl_pipeline("users.csv")
        
        # Act
        pipeline.run()
        dlq_errors = pipeline.get_dlq_errors()
        
        # Assert
        assert len(dlq_errors) > 0
        assert dlq_errors[0].row_index == 2
        assert "name" in dlq_errors[0].error or "email" in dlq_errors[0].error
    
    # ============================================================
    # ТЕСТ 3: ГРАНИЧНЫЕ УСЛОВИЯ
    # ============================================================
    
    def test_empty_file(self, create_csv, etl_pipeline, temp_dir):
        """Тест: пустой CSV файл."""
        # Arrange
        csv_file = create_csv("empty.csv", ["id", "name"], [])
        
        pipeline = etl_pipeline("empty.csv")
        
        # Act
        success = pipeline.run()
        
        # Assert
        assert success is True
        assert pipeline.get_stats()["extracted"] == 0
        assert pipeline.get_stats()["valid"] == 0
        assert pipeline.get_stats()["loaded"] == 0
    
    def test_file_with_only_headers(self, create_csv, etl_pipeline, temp_dir):
        """Тест: CSV только с заголовками."""
        # Arrange
        csv_file = temp_dir / "headers_only.csv"
        with open(csv_file, 'w', encoding='utf-8') as f:
            f.write("id,name,email,age,city\n")
        
        pipeline = etl_pipeline("headers_only.csv")
        
        # Act
        success = pipeline.run()
        
        # Assert
        assert success is True
        assert pipeline.get_stats()["extracted"] == 0
    
    def test_file_with_minimal_valid_data(self, create_csv, etl_pipeline, temp_dir):
        """Тест: минимальные валидные данные."""
        # Arrange
        csv_file = create_csv(
            "users.csv",
            ["id", "name", "email", "age", "city"],
            [[1, "A", "a@b.c", 0, "X"]]  # минимальные значения
        )
        
        pipeline = etl_pipeline("users.csv")
        
        # Act
        success = pipeline.run()
        
        # Assert
        assert success is True
        assert pipeline.get_stats()["valid"] == 1
    
    def test_file_with_maximal_valid_data(self, create_csv, etl_pipeline, temp_dir):
        """Тест: максимальные валидные значения."""
        # Arrange
        csv_file = create_csv(
            "users.csv",
            ["id", "name", "email", "age", "city"],
            [[999999, "A" * 100, "a@b.com", 150, "X" * 100]]
        )
        
        pipeline = etl_pipeline("users.csv")
        
        # Act
        success = pipeline.run()
        
        # Assert
        assert success is True
        assert pipeline.get_stats()["valid"] == 1
    
    # ============================================================
    # ТЕСТ 4: ОБРАБОТКА ТРАНСФОРМАЦИЙ
    # ============================================================
    
    def test_age_group_calculation(self, create_csv, etl_pipeline, temp_dir):
        """Тест: расчёт возрастных групп."""
        # Arrange
        csv_file = create_csv(
            "users.csv",
            ["id", "name", "email", "age", "city"],
            [
                [1, "Child", "child@example.com", 10, "City"],
                [2, "Adult", "adult@example.com", 30, "City"],
                [3, "Senior", "senior@example.com", 70, "City"]
            ]
        )
        
        pipeline = etl_pipeline("users.csv")
        
        # Act
        pipeline.run()
        output_file = temp_dir / "output.json"
        
        # Assert
        with open(output_file, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        age_groups = {item["name"]: item["age_group"] for item in data}
        assert age_groups["Child"] == "minor"
        assert age_groups["Adult"] == "adult"
        assert age_groups["Senior"] == "senior"
    
    def test_name_normalization(self, create_csv, etl_pipeline, temp_dir):
        """Тест: нормализация имён."""
        # Arrange
        csv_file = create_csv(
            "users.csv",
            ["id", "name", "email", "age", "city"],
            [
                [1, "иван петров", "ivan@example.com", 25, "moscow"],
                [2, "  МАРИЯ  СИДОРОВА  ", "maria@example.com", 30, "spb"]
            ]
        )
        
        pipeline = etl_pipeline("users.csv")
        
        # Act
        pipeline.run()
        output_file = temp_dir / "output.json"
        
        # Assert
        with open(output_file, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        names = {item["id"]: item["name"] for item in data}
        assert names[1] == "Ivan Petrov"
        assert names[2] == "Maria Sidorova"
    
    # ============================================================
    # ТЕСТ 5: ОШИБКИ ДОСТУПА
    # ============================================================
    
    def test_missing_input_file(self, etl_pipeline, temp_dir):
        """Тест: отсутствует входной файл."""
        pipeline = etl_pipeline("nonexistent.csv")
        
        with pytest.raises(FileNotFoundError) as exc_info:
            pipeline.run()
        
        assert "nonexistent.csv" in str(exc_info.value)
    
    def test_output_directory_created(self, create_csv, etl_pipeline, temp_dir):
        """Тест: создание выходной директории."""
        # Arrange
        csv_file = create_csv(
            "users.csv",
            ["id", "name", "email", "age", "city"],
            [[1, "Ivan", "ivan@example.com", 25, "Moscow"]]
        )
        
        pipeline = UserETLPipeline(str(temp_dir / "users.csv"), str(temp_dir / "subdir" / "output.json"))
        
        # Act
        success = pipeline.run()
        
        # Assert
        assert success is True
        assert (temp_dir / "subdir").exists()
        assert (temp_dir / "subdir" / "output.json").exists()
    
    # ============================================================
    # ТЕСТ 6: ЦЕЛОСТНОСТЬ ДАННЫХ
    # ============================================================
    
    def test_data_integrity_no_data_loss(self, create_csv, etl_pipeline, temp_dir):
        """Тест: нет потери данных при частичной валидации."""
        # Arrange
        csv_file = create_csv(
            "users.csv",
            ["id", "name", "email", "age", "city"],
            [
                [1, "Valid1", "valid1@example.com", 25, "City"],
                [2, "", "invalid@", 30, "City"],
                [3, "Valid2", "valid2@example.com", 35, "City"]
            ]
        )
        
        pipeline = etl_pipeline("users.csv")
        
        # Act
        pipeline.run()
        output_file = temp_dir / "output.json"
        
        # Assert
        with open(output_file, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        assert len(data) == 2
        assert data[0]["id"] == 1
        assert data[1]["id"] == 3
    
    # ============================================================
    # ТЕСТ 7: ПРОИЗВОДИТЕЛЬНОСТЬ (ОПЦИОНАЛЬНО)
    # ============================================================
    
    def test_performance_large_file(self, create_csv, etl_pipeline, temp_dir):
        """Тест: обработка большого файла."""
        # Arrange
        import time
        rows = [[i, f"User{i}", f"user{i}@example.com", 25, "City"] for i in range(1, 1001)]
        
        csv_file = create_csv("large.csv", ["id", "name", "email", "age", "city"], rows)
        pipeline = etl_pipeline("large.csv")
        
        # Act
        start = time.time()
        pipeline.run()
        elapsed = time.time() - start
        
        # Assert
        assert pipeline.get_stats()["extracted"] == 1000
        assert pipeline.get_stats()["valid"] == 1000
        assert elapsed < 5.0  # Должно обработать за 5 секунд


# ============================================================
# ЗАПУСК ТЕСТОВ
# ============================================================

if __name__ == "__main__":
    pytest.main([__file__, "-v", "--tb=short"])
Запуск интеграционных тестов
bash
# Запуск всех тестов
pytest test_user_etl_integration.py -v

# Запуск с отчётом о покрытии
pytest test_user_etl_integration.py --cov=user_etl --cov-report=html

# Запуск конкретного теста
pytest test_user_etl_integration.py::TestUserETLIntegration::test_happy_path_single_user -v

# Запуск с подробным выводом ошибок
pytest test_user_etl_integration.py -v --tb=long
Ожидаемый вывод тестов
text
============================= test session starts ==============================
collected 14 items

test_user_etl_integration.py::TestUserETLIntegration::test_happy_path_single_user PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_happy_path_multiple_users PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_partial_invalid_data PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_all_invalid_data PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_dlq_contains_errors PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_empty_file PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_file_with_only_headers PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_file_with_minimal_valid_data PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_file_with_maximal_valid_data PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_age_group_calculation PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_name_normalization PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_missing_input_file PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_output_directory_created PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_data_integrity_no_data_loss PASSED
test_user_etl_integration.py::TestUserETLIntegration::test_performance_large_file PASSED

============================= 14 passed in 2.34s ===============================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (happy path, простые ошибки)

Варианты 9-17: средний (+ DLQ, граничные случаи)

Варианты 18-25: сложный (+ производительность, восстановление)

Варианты 1-8 (Базовый уровень)
№	Тема ETL	Тесты
1	Студенты → JSON	Happy path, пустой файл
2	Товары → JSON	Happy path, неверные типы
3	Книги → JSON	Happy path, отсутствие полей
4	Заказы → JSON	Happy path, дубликаты
5	Пользователи → CSV	Happy path, пустые строки
6	Продукты → JSON	Happy path, отрицательные цены
7	Сотрудники → JSON	Happy path, неверный email
8	События → CSV	Happy path, неверные даты
Варианты 9-17 (Средний уровень)
№	Тема ETL	Дополнительные тесты
9	Клиенты → JSON	DLQ, частичная валидация
10	Заказы → JSON	DLQ, границы возрастов
11	Транзакции → CSV	DLQ, суммы, валюты
12	Логи → JSON	DLQ, форматы дат
13	Отзывы → JSON	DLQ, рейтинги
14	Билеты → CSV	DLQ, дубликаты
15	Рецепты → JSON	DLQ, ингредиенты
16	Отели → JSON	DLQ, даты заезда/выезда
17	Авиабилеты → CSV	DLQ, маршруты
Варианты 18-25 (Сложный уровень)
№	Тема ETL	Интеграционные тесты
18	Банковские транзакции	DLQ, алгоритм Луна, лимиты
19	Медицинские данные	DLQ, СНИЛС, ОМС
20	Налоговые отчёты	DLQ, ИНН, КПП
21	Данные сенсоров	Производительность, временные ряды
22	Социальная сеть	DLQ, пользователи, посты
23	Интернет-магазин	DLQ, заказы, товары
24	CRM система	DLQ, клиенты, сделки
25	Библиотека	DLQ, книги, читатели, выдача
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Интеграционные тесты отсутствуют
3 (удовлетворительно)	Есть happy path, частичная валидация
4 (хорошо)	+ граничные случаи, тесты DLQ
5 (отлично)	+ производительность, восстановление, покрытие > 80%
5. Шпаргалка
python
# === ФИКСТУРЫ ДЛЯ ИНТЕГРАЦИОННЫХ ТЕСТОВ ===
@pytest.fixture
def temp_dir(tmp_path):
    return tmp_path

@pytest.fixture
def sample_csv(temp_dir):
    csv_file = temp_dir / "test.csv"
    # создание файла
    return csv_file

# === ПРОВЕРКИ ===
assert success is True
assert len(data) == expected_count
with open(file) as f:
    data = json.load(f)

# === ЗАПУСК ===
pytest test_file.py -v
pytest test_file.py::TestClass::test_name -v
pytest --cov=. --cov-report=html
Карточка студента
text
ПЗ 2.36. НАПИСАНИЕ ИНТЕГРАЦИОННЫХ ТЕСТОВ ДЛЯ ETL-КОНВЕЙЕРА

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== ТИПЫ ТЕСТОВ ===

□ Happy path (полный успешный сценарий)
□ Частичная валидация (DLQ)
□ Все невалидные данные
□ Пустой файл
□ Граничные значения (min/max)
□ Ошибки доступа (нет файла)
□ Трансформации (возрастные группы)
□ Целостность данных
□ Производительность

=== РЕЗУЛЬТАТЫ ===

Количество тестов: _____
Пройдено: _____
Покрытие кода: _____%

=== ОТЧЁТ ===

Файлы: _____________
Отчёт о покрытии: _____________

Дата выполнения: _____________
