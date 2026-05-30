# ПЗ 2.3. Создание каркаса Python-пакета: модули и импорты

**Тема:** Структура Python-проекта, модули, пакеты, абсолютные и относительные импорты

**Цель работы:**  
Научиться создавать правильно организованный Python-пакет, использовать абсолютные и относительные импорты, применять `pathlib` для работы с файлами и понимать архитектурные шаблоны.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Git (опционально)

---

## Структура практической работы

| Часть | Название | Время |
|-------|----------|-------|
| 1 | Создание каркаса пакета | 20 мин |
| 2 | Реализация модулей с разными типами импортов | 25 мин |
| 3 | Тестирование импортов и работа с pathlib | 20 мин |
| 4 | Рефакторинг по архитектурному шаблону | 25 мин |

---

## Часть 1. Создание каркаса пакета (20 минут)

### Задание 1.1. Создайте структуру папок

В терминале VS Code выполните:

```bash
# Создаём корневую папку проекта
mkdir geometry_package
cd geometry_package

# Создаём основную структуру
mkdir -p src/geometry/models
mkdir -p src/geometry/calculators
mkdir -p src/geometry/utils
mkdir -p tests
mkdir -p data

# Создаём пустые файлы __init__.py
touch src/__init__.py
touch src/geometry/__init__.py
touch src/geometry/models/__init__.py
touch src/geometry/calculators/__init__.py
touch src/geometry/utils/__init__.py
touch tests/__init__.py

# Создаём основной файл приложения
touch main.py

# Создаём служебные файлы
touch requirements.txt
touch README.md
touch .gitignore
Для Windows (PowerShell):

powershell
# Создаём корневую папку проекта
mkdir geometry_package
cd geometry_package

# Создаём основную структуру
New-Item -ItemType Directory -Force -Path src/geometry/models
New-Item -ItemType Directory -Force -Path src/geometry/calculators
New-Item -ItemType Directory -Force -Path src/geometry/utils
New-Item -ItemType Directory -Force -Path tests
New-Item -ItemType Directory -Force -Path data

# Создаём пустые файлы __init__.py
New-Item -ItemType File -Force -Path src/__init__.py
New-Item -ItemType File -Force -Path src/geometry/__init__.py
New-Item -ItemType File -Force -Path src/geometry/models/__init__.py
New-Item -ItemType File -Force -Path src/geometry/calculators/__init__.py
New-Item -ItemType File -Force -Path src/geometry/utils/__init__.py
New-Item -ItemType File -Force -Path tests/__init__.py

# Создаём остальные файлы
New-Item -ItemType File -Force -Path main.py
New-Item -ItemType File -Force -Path requirements.txt
New-Item -ItemType File -Force -Path README.md
New-Item -ItemType File -Force -Path .gitignore
Задание 1.2. Проверьте структуру
Конечная структура должна выглядеть так:

text
geometry_package/
├── main.py
├── requirements.txt
├── README.md
├── .gitignore
├── src/
│   ├── __init__.py
│   └── geometry/
│       ├── __init__.py
│       ├── models/
│       │   └── __init__.py
│       ├── calculators/
│       │   └── __init__.py
│       └── utils/
│           └── __init__.py
├── tests/
│   └── __init__.py
└── data/
Что сдавать: скриншот дерева папок (команда tree или скриншот из VS Code).

Часть 2. Реализация модулей с разными типами импортов (25 минут)
Задание 2.1. Создайте модуль моделей
Создайте файл src/geometry/models/shapes.py:

python
"""
Модуль с моделями геометрических фигур.
Содержит классы для представления фигур.
"""

import math
from dataclasses import dataclass
from typing import Union


@dataclass
class Circle:
    """Модель круга."""
    radius: float
    
    def __post_init__(self):
        if self.radius <= 0:
            raise ValueError("Радиус должен быть положительным")
    
    @property
    def diameter(self) -> float:
        return self.radius * 2
    
    def __str__(self) -> str:
        return f"Круг(радиус={self.radius})"


@dataclass
class Rectangle:
    """Модель прямоугольника."""
    width: float
    height: float
    
    def __post_init__(self):
        if self.width <= 0 or self.height <= 0:
            raise ValueError("Стороны должны быть положительными")
    
    @property
    def is_square(self) -> bool:
        return self.width == self.height
    
    def __str__(self) -> str:
        return f"Прямоугольник({self.width}×{self.height})"


@dataclass
class Triangle:
    """Модель треугольника (по трём сторонам)."""
    a: float
    b: float
    c: float
    
    def __post_init__(self):
        if self.a <= 0 or self.b <= 0 or self.c <= 0:
            raise ValueError("Стороны должны быть положительными")
        
        # Проверка неравенства треугольника
        if (self.a + self.b <= self.c or 
            self.a + self.c <= self.b or 
            self.b + self.c <= self.a):
            raise ValueError("Треугольник с такими сторонами не существует")
    
    @property
    def perimeter(self) -> float:
        return self.a + self.b + self.c
    
    def __str__(self) -> str:
        return f"Треугольник({self.a}, {self.b}, {self.c})"
Задание 2.2. Создайте модуль калькуляторов
Создайте файл src/geometry/calculators/area_calculator.py:

python
"""
Модуль с калькуляторами площадей.
Использует относительные импорты для доступа к моделям.
"""

# ОТНОСИТЕЛЬНЫЙ ИМПОРТ (работает только внутри пакета)
from ..models.shapes import Circle, Rectangle, Triangle
import math


class AreaCalculator:
    """Калькулятор площади фигур."""
    
    @staticmethod
    def circle_area(circle: Circle) -> float:
        """Вычисляет площадь круга."""
        return math.pi * circle.radius ** 2
    
    @staticmethod
    def rectangle_area(rectangle: Rectangle) -> float:
        """Вычисляет площадь прямоугольника."""
        return rectangle.width * rectangle.height
    
    @staticmethod
    def triangle_area(triangle: Triangle) -> float:
        """Вычисляет площадь треугольника по формуле Герона."""
        p = triangle.perimeter / 2  # полупериметр
        return math.sqrt(p * (p - triangle.a) * (p - triangle.b) * (p - triangle.c))
    
    @staticmethod
    def calculate(shape: Union[Circle, Rectangle, Triangle]) -> float:
        """Универсальный метод вычисления площади."""
        if isinstance(shape, Circle):
            return AreaCalculator.circle_area(shape)
        elif isinstance(shape, Rectangle):
            return AreaCalculator.rectangle_area(shape)
        elif isinstance(shape, Triangle):
            return AreaCalculator.triangle_area(shape)
        else:
            raise TypeError(f"Неподдерживаемый тип фигуры: {type(shape)}")
Задание 2.3. Создайте модуль калькулятора периметров
Создайте файл src/geometry/calculators/perimeter_calculator.py:

python
"""
Модуль с калькуляторами периметров.
Демонстрирует альтернативный стиль импортов.
"""

# АБСОЛЮТНЫЙ ИМПОРТ (указывает полный путь)
from src.geometry.models.shapes import Circle, Rectangle, Triangle


class PerimeterCalculator:
    """Калькулятор периметра фигур."""
    
    @staticmethod
    def circle_perimeter(circle: Circle) -> float:
        """Вычисляет длину окружности."""
        return 2 * math.pi * circle.radius
    
    @staticmethod
    def rectangle_perimeter(rectangle: Rectangle) -> float:
        """Вычисляет периметр прямоугольника."""
        return 2 * (rectangle.width + rectangle.height)
    
    @staticmethod
    def triangle_perimeter(triangle: Triangle) -> float:
        """Вычисляет периметр треугольника."""
        return triangle.perimeter
    
    @staticmethod
    def calculate(shape: Union[Circle, Rectangle, Triangle]) -> float:
        """Универсальный метод вычисления периметра."""
        if isinstance(shape, Circle):
            return PerimeterCalculator.circle_perimeter(shape)
        elif isinstance(shape, Rectangle):
            return PerimeterCalculator.rectangle_perimeter(shape)
        elif isinstance(shape, Triangle):
            return PerimeterCalculator.triangle_perimeter(shape)
        else:
            raise TypeError(f"Неподдерживаемый тип фигуры: {type(shape)}")
Обратите внимание: в этом файле не хватает import math и from typing import Union. Исправьте эту ошибку самостоятельно!

Задание 2.4. Создайте модуль утилит
Создайте файл src/geometry/utils/validators.py:

python
"""
Модуль с утилитами для валидации.
Демонстрирует смешанные типы импортов.
"""

from typing import Any, Optional
import json
from pathlib import Path


def validate_positive(value: float, name: str = "Значение") -> float:
    """Проверяет, что число положительное."""
    if value <= 0:
        raise ValueError(f"{name} должно быть положительным, получено {value}")
    return value


def validate_shape_data(data: dict) -> bool:
    """Валидирует данные фигуры из JSON."""
    required_fields = ['type', 'params']
    
    for field in required_fields:
        if field not in data:
            return False
    
    shape_type = data['type']
    params = data['params']
    
    if shape_type == 'circle' and 'radius' in params:
        return params['radius'] > 0
    elif shape_type == 'rectangle' and 'width' in params and 'height' in params:
        return params['width'] > 0 and params['height'] > 0
    elif shape_type == 'triangle' and all(k in params for k in ['a', 'b', 'c']):
        a, b, c = params['a'], params['b'], params['c']
        return (a > 0 and b > 0 and c > 0 and 
                a + b > c and a + c > b and b + c > a)
    
    return False


def load_shapes_from_json(file_path: Path) -> list:
    """Загружает фигуры из JSON-файла."""
    if not file_path.exists():
        return []
    
    content = file_path.read_text(encoding='utf-8')
    data = json.loads(content)
    
    if isinstance(data, list):
        return data
    return []
Задание 2.5. Создайте __init__.py для удобного импорта
Отредактируйте файл src/geometry/__init__.py:

python
"""
Пакет geometry — работа с геометрическими фигурами.
Позволяет вычислять площади и периметры.
"""

# Экспортируем основные классы и функции для удобного импорта
from .models.shapes import Circle, Rectangle, Triangle
from .calculators.area_calculator import AreaCalculator
from .calculators.perimeter_calculator import PerimeterCalculator
from .utils.validators import validate_positive, validate_shape_data

# Определяем, что импортируется при `from geometry import *`
__all__ = [
    'Circle',
    'Rectangle', 
    'Triangle',
    'AreaCalculator',
    'PerimeterCalculator',
    'validate_positive',
    'validate_shape_data'
]
Часть 3. Тестирование импортов и работа с pathlib (20 минут)
Задание 3.1. Создайте главный модуль с разными типами импортов
Создайте файл main.py в корне проекта:

python
#!/usr/bin/env python3
"""
Главный модуль демонстрирует различные способы импорта:
1. Импорт всего пакета
2. Импорт конкретных классов
3. Импорт с псевдонимом
"""

# ============================================================
# Способ 1: импорт всего пакета (через __init__.py)
# ============================================================
import src.geometry as geom

# Создаём фигуры с помощью импортированных классов
circle = geom.Circle(radius=5)
rectangle = geom.Rectangle(width=4, height=6)
triangle = geom.Triangle(a=3, b=4, c=5)

# Вычисляем площади
print("=== Способ 1: импорт всего пакета ===")
print(f"Круг: {circle}")
print(f"  Площадь: {geom.AreaCalculator.circle_area(circle):.2f}")
print(f"  Периметр: {geom.PerimeterCalculator.circle_perimeter(circle):.2f}")

print(f"\nПрямоугольник: {rectangle}")
print(f"  Площадь: {geom.AreaCalculator.rectangle_area(rectangle):.2f}")
print(f"  Периметр: {geom.PerimeterCalculator.rectangle_perimeter(rectangle):.2f}")

print(f"\nТреугольник: {triangle}")
print(f"  Площадь: {geom.AreaCalculator.triangle_area(triangle):.2f}")
print(f"  Периметр: {geom.PerimeterCalculator.triangle_perimeter(triangle):.2f}")

# ============================================================
# Способ 2: импорт конкретных классов
# ============================================================
from src.geometry.calculators.area_calculator import AreaCalculator
from src.geometry.calculators.perimeter_calculator import PerimeterCalculator
from src.geometry.models.shapes import Rectangle as Rect

print("\n=== Способ 2: импорт конкретных классов ===")
rect = Rect(width=10, height=5)
print(f"Прямоугольник 10×5: площадь = {AreaCalculator.rectangle_area(rect)}")
print(f"Периметр = {PerimeterCalculator.rectangle_perimeter(rect)}")

# ============================================================
# Способ 3: импорт с использованием псевдонимов
# ============================================================
from src.geometry.utils.validators import validate_positive as vp

print("\n=== Способ 3: импорт с псевдонимом ===")
try:
    vp(-5, "Радиус")
except ValueError as e:
    print(f"Валидация сработала: {e}")

# ============================================================
# Работа с pathlib: сохранение результатов
# ============================================================
from pathlib import Path
import json
from datetime import datetime

def save_results_to_json(shapes_data: list, filename: str = "results.json"):
    """Сохраняет результаты вычислений в JSON-файл (кроссплатформенно)."""
    # Создаём папку data, если её нет
    data_dir = Path(__file__).parent / "data"
    data_dir.mkdir(exist_ok=True)
    
    # Формируем полный путь к файлу
    file_path = data_dir / filename
    
    # Добавляем временную метку
    output = {
        "timestamp": datetime.now().isoformat(),
        "results": shapes_data
    }
    
    # Сохраняем
    file_path.write_text(json.dumps(output, indent=2, ensure_ascii=False), encoding='utf-8')
    print(f"\nРезультаты сохранены в: {file_path.absolute()}")
    return file_path

# Сохраняем результаты
results = [
    {"shape": "Круг", "radius": 5, "area": 78.54, "perimeter": 31.42},
    {"shape": "Прямоугольник", "width": 4, "height": 6, "area": 24.0, "perimeter": 20.0},
    {"shape": "Треугольник", "sides": [3, 4, 5], "area": 6.0, "perimeter": 12.0}
]

save_results_to_json(results, "shapes_results.json")

print("\n✓ Все импорты работают корректно!")
Задание 3.2. Запустите и проверьте
bash
# Убедитесь, что находитесь в корне проекта geometry_package
cd geometry_package

# Запустите main.py
python main.py
Ожидаемый вывод:

text
=== Способ 1: импорт всего пакета ===
Круг: Круг(радиус=5)
  Площадь: 78.54
  Периметр: 31.42

Прямоугольник: Прямоугольник(4×6)
  Площадь: 24.00
  Периметр: 20.00

Треугольник: Треугольник(3, 4, 5)
  Площадь: 6.00
  Периметр: 12.00

=== Способ 2: импорт конкретных классов ===
Прямоугольник 10×5: площадь = 50.0
Периметр = 30.0

=== Способ 3: импорт с псевдонимом ===
Валидация сработала: Радиус должно быть положительным, получено -5

Результаты сохранены в: /путь/к/geometry_package/data/shapes_results.json

✓ Все импорты работают корректно!
Задание 3.3. Эксперименты с импортами (выполните в интерактивном режиме)
Запустите Python в терминале и выполните следующие эксперименты:

bash
# Запускаем интерактивный режим из корня проекта
python
python
# Эксперимент 1: импорт через __init__.py
>>> import src.geometry as geom
>>> geom.Circle(10)
Круг(радиус=10)

# Эксперимент 2: импорт подмодуля
>>> from src.geometry.models import shapes
>>> shapes.Rectangle(5, 10)
Прямоугольник(5×10)

# Эксперимент 3: относительный импорт НЕ работает из main
# (этот код вызовет ошибку, если его запустить напрямую)
>>> from .models.shapes import Circle
ImportError: attempted relative import with no known parent package
Что сдавать: скриншот успешного выполнения main.py и результаты экспериментов.

Часть 4. Рефакторинг по архитектурному шаблону (25 минут)
Задание 4.1. Добавьте слой сервисов
Создайте файл src/geometry/services/shape_service.py:

python
"""
Сервисный слой — бизнес-логика приложения.
Использует репозиторий (который создадим позже) для работы с данными.
"""

from pathlib import Path
from typing import List, Optional, Dict, Any
from ..models.shapes import Circle, Rectangle, Triangle
from ..calculators.area_calculator import AreaCalculator
from ..calculators.perimeter_calculator import PerimeterCalculator
from ..utils.validators import validate_shape_data, load_shapes_from_json


class ShapeService:
    """
    Сервис для работы с геометрическими фигурами.
    Содержит бизнес-логику приложения.
    """
    
    def __init__(self, data_path: Optional[Path] = None):
        self.data_path = data_path or Path("./data/shapes.json")
        self.area_calc = AreaCalculator()
        self.perimeter_calc = PerimeterCalculator()
    
    def create_shape_from_dict(self, data: Dict[str, Any]) -> Optional[Circle | Rectangle | Triangle]:
        """Создаёт фигуру из словаря."""
        if not validate_shape_data(data):
            return None
        
        shape_type = data['type']
        params = data['params']
        
        if shape_type == 'circle':
            return Circle(radius=params['radius'])
        elif shape_type == 'rectangle':
            return Rectangle(width=params['width'], height=params['height'])
        elif shape_type == 'triangle':
            return Triangle(a=params['a'], b=params['b'], c=params['c'])
        
        return None
    
    def get_shape_info(self, shape: Circle | Rectangle | Triangle) -> Dict[str, Any]:
        """Возвращает полную информацию о фигуре."""
        return {
            "type": type(shape).__name__,
            "str": str(shape),
            "area": self.area_calc.calculate(shape),
            "perimeter": self.perimeter_calc.calculate(shape)
        }
    
    def load_shapes_from_json(self) -> List[Dict[str, Any]]:
        """Загружает фигуры из JSON-файла."""
        return load_shapes_from_json(self.data_path)
    
    def process_json_shapes(self) -> List[Dict[str, Any]]:
        """Обрабатывает все фигуры из JSON и возвращает их параметры."""
        shapes_data = self.load_shapes_from_json()
        results = []
        
        for data in shapes_data:
            shape = self.create_shape_from_dict(data)
            if shape:
                results.append(self.get_shape_info(shape))
            else:
                results.append({"error": "Invalid shape data", "data": data})
        
        return results
Задание 4.2. Создайте пример данных для тестирования
Создайте файл data/shapes.json:

json
[
    {
        "type": "circle",
        "params": {
            "radius": 10
        }
    },
    {
        "type": "rectangle",
        "params": {
            "width": 5,
            "height": 8
        }
    },
    {
        "type": "triangle",
        "params": {
            "a": 6,
            "b": 8,
            "c": 10
        }
    },
    {
        "type": "circle",
        "params": {
            "radius": 2.5
        }
    }
]
Задание 4.3. Обновите main.py для демонстрации сервисного слоя
Добавьте в конец файла main.py:

python
# ============================================================
# Часть 4: Использование сервисного слоя
# ============================================================
print("\n=== Часть 4: Сервисный слой ===")

from src.geometry.services.shape_service import ShapeService

# Создаём сервис с указанием пути к файлу данных
service = ShapeService(data_path=Path("./data/shapes.json"))

# Обрабатываем фигуры из JSON
results = service.process_json_shapes()

for i, result in enumerate(results, 1):
    if "error" in result:
        print(f"{i}. ОШИБКА: {result['error']}")
    else:
        print(f"{i}. {result['type']}: {result['str']}")
        print(f"   Площадь: {result['area']:.2f}, Периметр: {result['perimeter']:.2f}")

# Сохраняем результаты обработки
save_results_to_json(results, "processed_shapes.json")
Задание 4.4. Запустите обновлённый проект
bash
python main.py
Контрольные вопросы для самопроверки
Ответьте письменно (2-3 предложения на вопрос):

В чём разница между абсолютным и относительным импортом? Приведите примеры из выполненной работы.

Почему в файле perimeter_calculator.py возникла ошибка и как её исправить?

Что произойдёт, если удалить файл src/geometry/__init__.py? Проверьте экспериментально.

Как с помощью pathlib создать вложенную папку a/b/c/d одной командой?

Какой архитектурный шаблон реализован в части 4? Назовите его основные компоненты.

Что сдавать
В отчёт по ПЗ 2.3 включить:

Скриншот структуры проекта (дерево папок).

Содержимое всех созданных файлов (можно текстом или скриншотами):

src/geometry/__init__.py

src/geometry/models/shapes.py

src/geometry/calculators/area_calculator.py

src/geometry/calculators/perimeter_calculator.py (с исправленной ошибкой)

src/geometry/utils/validators.py

src/geometry/services/shape_service.py

main.py

Скриншот вывода программы (результат выполнения python main.py).

Содержимое созданного JSON-файла data/processed_shapes.json.

Ответы на контрольные вопросы.

Критерии оценки
Баллы	Критерий
3 (зачтено)	Создана структура пакета, работают базовые импорты
4 (хорошо)	+ Реализованы все модули, исправлена ошибка в perimeter_calculator.py
5 (отлично)	+ Реализован сервисный слой, программа выводит корректные результаты
Шпаргалка по типам импортов
python
# === АБСОЛЮТНЫЕ ИМПОРТЫ (от корня проекта) ===
import src.geometry.models.shapes
from src.geometry.calculators.area_calculator import AreaCalculator
from src.geometry.utils.validators import validate_positive as vp

# === ОТНОСИТЕЛЬНЫЕ ИМПОРТЫ (внутри пакета) ===
from .models.shapes import Circle          # из текущего пакета
from ..calculators.area_calculator import AreaCalculator  # на уровень выше
from ...utils.helpers import helper_func   # на два уровня выше

# === ИМПОРТ ЧЕРЕЗ __init__.py ===
import src.geometry as geom
geom.Circle(10)
geom.AreaCalculator.circle_area(circle)

# === ДИНАМИЧЕСКИЕ ИМПОРТЫ (редко, но полезно) ===
module = __import__('src.geometry.models.shapes')
Итог практической работы
Вы сегодня:

✅ Создали полноценный Python-пакет с правильной структурой

✅ Научились использовать абсолютные и относительные импорты

✅ Исправили типичную ошибку с отсутствующим импортом

✅ Применили pathlib для кроссплатформенной работы с файлами

✅ Реализовали архитектурный шаблон "Сервис-Модель-Утилиты"

✅ Сохранили результаты работы в JSON с помощью pathlib

Теперь вы готовы создавать профессионально организованные Python-проекты!
