# ПЗ 2.3. 25 ВАРИАНТОВ ПРАКТИЧЕСКОЙ РАБОТЫ
## Тема: Создание каркаса Python-пакета: модули и импорты

---

## Нулевой вариант (эталонный) — Демонстрация преподавателя

**Тема проекта:** Геометрический калькулятор

**Структура пакета:**
geometry_package/
├── main.py
├── src/
│ └── geometry/
│ ├── init.py
│ ├── models/
│ │ ├── init.py
│ │ └── shapes.py
│ ├── calculators/
│ │ ├── init.py
│ │ ├── area.py
│ │ └── perimeter.py
│ └── utils/
│ ├── init.py
│ └── validators.py
├── tests/
└── data/

text

**Типы импортов для демонстрации:**

| Файл | Тип импорта | Пример |
|------|-------------|--------|
| `calculators/area.py` | Относительный | `from ..models.shapes import Circle` |
| `calculators/perimeter.py` | Абсолютный | `from src.geometry.models.shapes import Rectangle` |
| `main.py` | Через `__init__.py` | `import src.geometry as geom` |
| `utils/validators.py` | С псевдонимом | `from pathlib import Path as P` |

---

## Варианты 1-5 (Базовый уровень — Калькуляторы)

### Вариант 1. Калькулятор длины слов

**Тема проекта:** Анализатор текста

**Структура пакета:**
text_analyzer/
├── main.py
├── src/
│ └── textools/
│ ├── init.py
│ ├── models/
│ │ ├── init.py
│ │ └── word.py
│ ├── calculators/
│ │ ├── init.py
│ │ └── length_calc.py
│ └── utils/
│ ├── init.py
│ └── cleaners.py
└── data/

text

**Задание:**
1. Создайте модуль `word.py` с классом `Word` (атрибуты: `text`, `length`)
2. Создайте модуль `length_calc.py` с функцией `average_length()`
3. В `length_calc.py` используйте **относительный импорт** для `Word`
4. В `main.py` используйте **абсолютный импорт**
5. Создайте `__init__.py`, экспортирующий `Word` и `average_length`

---

### Вариант 2. Конвертер валют

**Тема проекта:** Финансовый конвертер

**Структура пакета:**
currency_converter/
├── main.py
├── src/
│ └── converter/
│ ├── init.py
│ ├── models/
│ │ ├── init.py
│ │ └── currency.py
│ ├── services/
│ │ ├── init.py
│ │ └── exchange.py
│ └── utils/
│ ├── init.py
│ └── rates.py
└── data/

text

**Задание:**
1. Создайте модуль `currency.py` с классом `Currency` (код, курс к рублю)
2. Создайте модуль `exchange.py` с функцией `convert()`
3. В `exchange.py` используйте **абсолютный импорт** для `Currency`
4. В `main.py` используйте **импорт с псевдонимом** (`import ... as conv`)
5. В `utils/rates.py` используйте **относительный импорт** на уровень выше

---

### Вариант 3. Генератор паролей

**Тема проекта:** Безопасность

**Структура пакета:**
password_gen/
├── main.py
├── src/
│ └── pwdgen/
│ ├── init.py
│ ├── core/
│ │ ├── init.py
│ │ └── generator.py
│ ├── validators/
│ │ ├── init.py
│ │ └── strength.py
│ └── utils/
│ ├── init.py
│ └── characters.py
└── data/

text

**Задание:**
1. Создайте модуль `generator.py` с классом `PasswordGenerator`
2. Создайте модуль `strength.py` с функцией `check_strength()`
3. В `generator.py` используйте **относительный импорт** из `utils.characters`
4. В `main.py` используйте `from src.pwdgen.core import PasswordGenerator`
5. В `characters.py` используйте **абсолютный импорт** из `string` (стандартная библиотека)

---

### Вариант 4. Калькулятор калорий

**Тема проекта:** Здоровое питание

**Структура пакета:**
calorie_calc/
├── main.py
├── src/
│ └── nutrition/
│ ├── init.py
│ ├── models/
│ │ ├── init.py
│ │ └── food.py
│ ├── calculators/
│ │ ├── init.py
│ │ └── calorie_counter.py
│ └── db/
│ ├── init.py
│ └── food_data.py
└── data/

text

**Задание:**
1. Создайте модуль `food.py` с классом `Food` (название, калории на 100г)
2. Создайте модуль `calorie_counter.py` с функцией `calculate()`
3. В `calorie_counter.py` используйте **относительный импорт** из `models`
4. В `food_data.py` используйте **абсолютный импорт** из `json` и `pathlib`
5. В `__init__.py` экспортируйте только `Food` и `calculate`

---

### Вариант 5. Конвертер температур

**Тема проекта:** Физические величины

**Структура пакета:**
temp_converter/
├── main.py
├── src/
│ └── temps/
│ ├── init.py
│ ├── scales/
│ │ ├── init.py
│ │ └── temperature.py
│ ├── converters/
│ │ ├── init.py
│ │ └── convert.py
│ └── formulas/
│ ├── init.py
│ └── equations.py
└── data/

text

**Задание:**
1. Создайте модуль `temperature.py` с классом `Temperature`
2. Создайте модуль `convert.py` с функциями `to_celsius()`, `to_fahrenheit()`
3. В `convert.py` используйте **относительный импорт** из `scales`
4. В `equations.py` используйте **абсолютный импорт** из `math`
5. В `main.py` используйте `from src.temps import Temperature`

---

## Варианты 6-10 (Средний уровень — Работа с данными)

### Вариант 6. Менеджер задач (To-Do)

**Тема проекта:** Продуктивность

**Структура пакета:**
todo_manager/
├── main.py
├── src/
│ └── todo/
│ ├── init.py
│ ├── models/
│ │ ├── init.py
│ │ └── task.py
│ ├── repositories/
│ │ ├── init.py
│ │ └── task_repo.py
│ ├── services/
│ │ ├── init.py
│ │ └── task_service.py
│ └── utils/
│ ├── init.py
│ └── formatters.py
└── data/

text

**Задание:**
1. Создайте `task.py` с классом `Task` (id, title, completed, created_at)
2. Создайте `task_repo.py` с классом `TaskRepository` (save, find, delete)
3. Создайте `task_service.py` с классом `TaskService` (add, complete, list)
4. В `task_repo.py` используйте **относительный импорт** из `models`
5. В `task_service.py` используйте **абсолютный импорт** из `repositories`
6. В `__init__.py` экспортируйте `Task` и `TaskService`

---

### Вариант 7. Контактная книга

**Тема проекта:** Управление контактами

**Структура пакета:**
contact_book/
├── main.py
├── src/
│ └── contacts/
│ ├── init.py
│ ├── entities/
│ │ ├── init.py
│ │ └── contact.py
│ ├── storage/
│ │ ├── init.py
│ │ └── json_storage.py
│ ├── handlers/
│ │ ├── init.py
│ │ └── contact_handler.py
│ └── validators/
│ ├── init.py
│ └── email_validator.py
└── data/

text

**Задание:**
1. Создайте `contact.py` с классом `Contact` (name, email, phone)
2. Создайте `json_storage.py` с классом `JSONStorage`
3. Создайте `contact_handler.py` с классом `ContactHandler`
4. В `json_storage.py` используйте **абсолютный импорт** `pathlib`
5. В `contact_handler.py` используйте **относительные импорты** из `entities` и `storage`
6. В `email_validator.py` используйте **импорт с псевдонимом** `re` как `regex`

---

### Вариант 8. Бюджетный трекер

**Тема проекта:** Личные финансы

**Структура пакета:**
budget_tracker/
├── main.py
├── src/
│ └── budget/
│ ├── init.py
│ ├── models/
│ │ ├── init.py
│ │ ├── transaction.py
│ │ └── category.py
│ ├── analytics/
│ │ ├── init.py
│ │ └── stats.py
│ └── exporters/
│ │ ├── init.py
│ │ └── csv_exporter.py
│ └── parsers/
│ ├── init.py
│ └── date_parser.py
└── data/

text

**Задание:**
1. Создайте `transaction.py` с классом `Transaction`
2. Создайте `category.py` с классом `Category` и `enum`
3. Создайте `stats.py` с функциями `total()`, `by_category()`
4. В `stats.py` используйте **относительные импорты** из `models`
5. В `csv_exporter.py` используйте **абсолютный импорт** `csv` и `pathlib`
6. В `__init__.py` настройте `__all__ = ['Transaction', 'Category', 'total']`

---

### Вариант 9. Библиотека книг

**Тема проекта:** Каталогизация

**Структура пакета:**
library/
├── main.py
├── src/
│ └── library/
│ ├── init.py
│ ├── domain/
│ │ ├── init.py
│ │ ├── book.py
│ │ └── author.py
│ ├── services/
│ │ ├── init.py
│ │ └── search_service.py
│ └── utils/
│ ├── init.py
│ └── isbn_validator.py
└── data/

text

**Задание:**
1. Создайте `book.py` с классом `Book` (title, author_id, isbn, year)
2. Создайте `author.py` с классом `Author` (name, birth_year)
3. Создайте `search_service.py` с классом `SearchService`
4. В `search_service.py` используйте **абсолютные импорты** из `domain`
5. В `isbn_validator.py` используйте **относительный импорт** (нет, он один в пакете)
6. В `main.py` используйте `import src.library as lib`

---

### Вариант 10. Рецепты и ингредиенты

**Тема проекта:** Кулинария

**Структура пакета:**
recipe_book/
├── main.py
├── src/
│ └── recipes/
│ ├── init.py
│ ├── data/
│ │ ├── init.py
│ │ └── recipe.py
│ ├── logic/
│ │ ├── init.py
│ │ └── recipe_calculator.py
│ └── io/
│ ├── init.py
│ └── recipe_loader.py
└── data/

text

**Задание:**
1. Создайте `recipe.py` с классом `Recipe` и вложенным классом `Ingredient`
2. Создайте `recipe_calculator.py` с функцией `scale_recipe()`
3. Создайте `recipe_loader.py` с функцией `load_from_json()`
4. В `recipe_loader.py` используйте **абсолютный импорт** из `data`
5. В `recipe_calculator.py` используйте **относительный импорт** на уровень выше
6. Создайте `__init__.py`, который импортирует `Recipe` и `scale_recipe`

---

## Варианты 11-15 (Продвинутый уровень — API и внешние данные)

### Вариант 11. Погодный клиент

**Тема проекта:** Работа с API

**Структура пакета:**
weather_client/
├── main.py
├── src/
│ └── weather/
│ ├── init.py
│ ├── api/
│ │ ├── init.py
│ │ └── openweather.py
│ ├── models/
│ │ ├── init.py
│ │ └── weather_data.py
│ ├── cache/
│ │ ├── init.py
│ │ └── file_cache.py
│ └── formatters/
│ ├── init.py
│ └── display.py
└── data/

text

**Задание:**
1. Создайте `weather_data.py` с классом `WeatherData`
2. Создайте `openweather.py` с классом `OpenWeatherClient`
3. Создайте `file_cache.py` с классом `FileCache`
4. В `openweather.py` используйте **абсолютные импорты** для `requests` и `json`
5. В `file_cache.py` используйте **относительный импорт** из `models`
6. В `display.py` используйте **относительный импорт** из `models` с псевдонимом

---

### Вариант 12. Клиент для цитат (Quotes API)

**Тема проекта:** Работа с REST API

**Структура пакета:**
quote_client/
├── main.py
├── src/
│ └── quotes/
│ ├── init.py
│ ├── clients/
│ │ ├── init.py
│ │ └── quote_client.py
│ ├── models/
│ │ ├── init.py
│ │ └── quote.py
│ ├── parsers/
│ │ ├── init.py
│ │ └── response_parser.py
│ └── writers/
│ ├── init.py
│ └── file_writer.py
└── data/

text

**Задание:**
1. Создайте `quote.py` с классом `Quote` (text, author, tags)
2. Создайте `quote_client.py` с классом `QuoteClient`
3. Создайте `response_parser.py` с функцией `parse_quote()`
4. В `quote_client.py` используйте **абсолютный импорт** `requests`
5. В `response_parser.py` используйте **относительный импорт** из `models`
6. В `file_writer.py` используйте **абсолютный импорт** `pathlib` с псевдонимом `P`

---

### Вариант 13. Генератор QR-кодов

**Тема проекта:** Визуализация данных

**Структура пакета:**
qr_generator/
├── main.py
├── src/
│ └── qrmaker/
│ ├── init.py
│ ├── core/
│ │ ├── init.py
│ │ └── generator.py
│ ├── renderers/
│ │ ├── init.py
│ │ └── image_renderer.py
│ ├── validators/
│ │ ├── init.py
│ │ └── input_validator.py
│ └── savers/
│ ├── init.py
│ └── file_saver.py
└── output/

text

**Задание:**
1. Создайте `generator.py` с классом `QRGenerator`
2. Создайте `image_renderer.py` с классом `ImageRenderer`
3. Создайте `input_validator.py` с функцией `validate_text()`
4. В `generator.py` используйте **относительные импорты** из `renderers` и `validators`
5. В `file_saver.py` используйте **абсолютный импорт** `pathlib`
6. В `__init__.py` экспортируйте только `QRGenerator` и `ImageRenderer`

---

### Вариант 14. Парсер вакансий (HeadHunter)

**Тема проекта:** Парсинг данных

**Структура пакета:**
job_parser/
├── main.py
├── src/
│ └── jobs/
│ ├── init.py
│ ├── fetchers/
│ │ ├── init.py
│ │ └── hh_fetcher.py
│ ├── models/
│ │ ├── init.py
│ │ └── vacancy.py
│ ├── filters/
│ │ ├── init.py
│ │ └── salary_filter.py
│ └── exporters/
│ ├── init.py
│ └── json_exporter.py
└── data/

text

**Задание:**
1. Создайте `vacancy.py` с классом `Vacancy`
2. Создайте `hh_fetcher.py` с классом `HHFetcher`
3. Создайте `salary_filter.py` с функцией `filter_by_salary()`
4. В `hh_fetcher.py` используйте **абсолютные импорты** (`requests`, `json`)
5. В `salary_filter.py` используйте **относительный импорт** из `models`
6. В `__init__.py` настройте импорт `Vacancy` и `HHFetcher`

---

### Вариант 15. Конвертер валют с кэшированием

**Тема проекта:** Кэширование данных

**Структура пакета:**
currency_cache/
├── main.py
├── src/
│ └── currency/
│ ├── init.py
│ ├── api/
│ │ ├── init.py
│ │ └── exchangerate.py
│ ├── cache/
│ │ ├── init.py
│ │ └── redis_cache.py
│ ├── converters/
│ │ ├── init.py
│ │ └── money_converter.py
│ └── models/
│ ├── init.py
│ └── rate.py
└── data/

text

**Задание:**
1. Создайте `rate.py` с классом `ExchangeRate`
2. Создайте `exchangerate.py` с классом `ExchangeRateAPI`
3. Создайте `redis_cache.py` с классом `FileBasedCache`
4. В `exchangerate.py` используйте **абсолютный импорт** из `cache`
5. В `money_converter.py` используйте **относительные импорты** из `api` и `cache`
6. В `__init__.py` используйте **относительный импорт** для `ExchangeRate`

---

## Варианты 16-20 (Экспертный уровень — Многослойная архитектура)

### Вариант 16. Система аутентификации

**Тема проекта:** Безопасность и авторизация

**Структура пакета:**
auth_system/
├── main.py
├── src/
│ └── auth/
│ ├── init.py
│ ├── entities/
│ │ ├── init.py
│ │ ├── user.py
│ │ └── role.py
│ ├── repositories/
│ │ ├── init.py
│ │ └── user_repo.py
│ ├── services/
│ │ ├── init.py
│ │ ├── auth_service.py
│ │ └── hash_service.py
│ ├── dto/
│ │ ├── init.py
│ │ └── login_dto.py
│ └── middleware/
│ ├── init.py
│ └── session.py
└── data/

text

**Задание:**
1. Создайте `user.py` и `role.py` с классами `User`, `Role`
2. Создайте `user_repo.py` с классом `UserRepository`
3. Создайте `auth_service.py` с классом `AuthService`
4. В `auth_service.py` используйте **абсолютные импорты** из `repositories`
5. В `hash_service.py` используйте **относительный импорт** из `entities`
6. В `__init__.py` экспортируйте `User`, `Role`, `AuthService`

---

### Вариант 17. Система логирования

**Тема проекта:** Мониторинг и отладка

**Структура пакета:**
logger_system/
├── main.py
├── src/
│ └── logger/
│ ├── init.py
│ ├── handlers/
│ │ ├── init.py
│ │ ├── file_handler.py
│ │ └── console_handler.py
│ ├── formatters/
│ │ ├── init.py
│ │ └── json_formatter.py
│ ├── levels/
│ │ ├── init.py
│ │ └── log_level.py
│ └── core/
│ ├── init.py
│ └── logger.py
└── logs/

text

**Задание:**
1. Создайте `log_level.py` с `Enum` уровней логирования
2. Создайте `file_handler.py` и `console_handler.py`
3. Создайте `logger.py` с классом `Logger`
4. В `logger.py` используйте **относительные импорты** из `handlers`, `formatters`
5. В `json_formatter.py` используйте **абсолютный импорт** `json`
6. В `__init__.py` экспортируйте `Logger` и `LogLevel`

---

### Вариант 18. Генератор отчётов

**Тема проекта:** Бизнес-аналитика

**Структура пакета:**
report_generator/
├── main.py
├── src/
│ └── reports/
│ ├── init.py
│ ├── collectors/
│ │ ├── init.py
│ │ └── data_collector.py
│ ├── processors/
│ │ ├── init.py
│ │ └── data_processor.py
│ ├── templates/
│ │ ├── init.py
│ │ └── template_engine.py
│ ├── exporters/
│ │ ├── init.py
│ │ ├── pdf_exporter.py
│ │ └── html_exporter.py
│ └── models/
│ ├── init.py
│ └── report.py
└── output/

text

**Задание:**
1. Создайте `report.py` с классом `Report`
2. Создайте `data_collector.py` и `data_processor.py`
3. Создайте `pdf_exporter.py` и `html_exporter.py`
4. В `data_processor.py` используйте **относительный импорт** из `collectors`
5. В `pdf_exporter.py` используйте **абсолютный импорт** из `models`
6. В `__init__.py` экспортируйте `Report` и `PDFExporter`

---

### Вариант 19. Система плагинов

**Тема проекта:** Расширяемая архитектура

**Структура пакета:**
plugin_system/
├── main.py
├── src/
│ └── plugins/
│ ├── init.py
│ ├── core/
│ │ ├── init.py
│ │ ├── plugin_manager.py
│ │ └── plugin_interface.py
│ ├── loader/
│ │ ├── init.py
│ │ └── module_loader.py
│ ├── registry/
│ │ ├── init.py
│ │ └── plugin_registry.py
│ └── builtin/
│ ├── init.py
│ ├── plugin_a.py
│ └── plugin_b.py
└── plugins/

text

**Задание:**
1. Создайте `plugin_interface.py` с абстрактным классом `Plugin`
2. Создайте `plugin_manager.py` с классом `PluginManager`
3. Создайте `module_loader.py` с функцией `load_plugin_from_path()`
4. В `plugin_manager.py` используйте **абсолютные импорты** из `loader` и `registry`
5. В `builtin/plugin_a.py` используйте **относительный импорт** из `core`
6. В `__init__.py` экспортируйте `Plugin` и `PluginManager`

---

### Вариант 20. Очередь задач (Task Queue)

**Тема проекта:** Асинхронная обработка

**Структура пакета:**
task_queue/
├── main.py
├── src/
│ └── queue/
│ ├── init.py
│ ├── models/
│ │ ├── init.py
│ │ └── task.py
│ ├── storage/
│ │ ├── init.py
│ │ └── queue_storage.py
│ ├── workers/
│ │ ├── init.py
│ │ └── worker.py
│ ├── dispatcher/
│ │ ├── init.py
│ │ └── dispatcher.py
│ └── serializers/
│ ├── init.py
│ └── json_serializer.py
└── data/

text

**Задание:**
1. Создайте `task.py` с классом `Task` (id, type, payload, status)
2. Создайте `queue_storage.py` с классом `QueueStorage`
3. Создайте `worker.py` с классом `Worker`
4. В `worker.py` используйте **относительные импорты** из `models` и `storage`
5. В `dispatcher.py` используйте **абсолютный импорт** из `workers`
6. В `__init__.py` используйте **относительный импорт** для `Task` и `Worker`

---

## Варианты 21-25 (Специальные — Исправление ошибок, альтернативные структуры)

### Вариант 21. Исправление ошибок импортов (рефакторинг)

**Дана структура с ошибками:**
broken_project/
├── main.py
├── src/
│ └── broken/
│ ├── init.py (пустой — плохо)
│ ├── a.py
│ └── b.py

text

**Файл `a.py`:**
```python
from .b import helper  # Относительный импорт вне пакета? Ошибка!

def func():
    return helper()
Файл b.py:

python
def helper():
    return "OK"
Файл main.py:

python
from src.broken.a import func
print(func())
Задание:

Проанализируйте, почему код не работает

Исправьте структуру и импорты так, чтобы main.py выводил "OK"

Добавьте файлы __init__.py с правильными экспортами

Создайте альтернативную версию с абсолютными импортами

Вариант 22. Циклический импорт (и исправление)
Дана структура с циклическим импортом:

text
cyclic_project/
├── main.py
└── src/
    └── cyclic/
        ├── __init__.py
        ├── module_a.py
        └── module_b.py
module_a.py:

python
from src.cyclic.module_b import function_b

def function_a():
    return "A -> " + function_b()
module_b.py:

python
from src.cyclic.module_a import function_a

def function_b():
    return "B -> " + function_a()
main.py:

python
from src.cyclic.module_a import function_a
print(function_a())
Задание:

Запустите код и объясните ошибку

Исправьте проблему, не меняя логику работы

Предложите 2 способа решения (рефакторинг, ленивый импорт)

Создайте правильную версию проекта

Вариант 23. Пакет с динамическими импортами
Структура проекта:

text
dynamic_imports/
├── main.py
├── src/
│   └── dynamic/
│       ├── __init__.py
│       ├── plugins/
│       │   ├── __init__.py
│       │   ├── plugin_one.py
│       │   └── plugin_two.py
│       └── loader.py
└── config.json
Задание:

Создайте два плагина: plugin_one.py с функцией execute() возвращает "One"

plugin_two.py с функцией execute() возвращает "Two"

В loader.py напишите функцию load_plugin(name), которая:

Использует importlib.import_module()

Динамически импортирует плагин по имени

Возвращает функцию execute

В main.py прочитайте из config.json имя плагина и выполните его

Используйте pathlib для чтения конфига

Вариант 24. Монолит в пакет (рефакторинг большого файла)
Дан монолитный файл monolith.py (500+ строк):

python
# monolith.py - всё в одном файле
import json, csv, re, sys, os
from pathlib import Path
from datetime import datetime

# Классы и функции для работы с пользователями
class User: ...
class Admin(User): ...
def create_user(): ...
def find_user(): ...
def validate_email(): ...

# Классы и функции для работы с продуктами
class Product: ...
class Cart: ...
def add_to_cart(): ...
def calculate_total(): ...

# Классы и функции для работы с заказами
class Order: ...
def create_order(): ...
def send_invoice(): ...

# UI и main
def display_menu(): ...
def main(): ...
Задание:

Разбейте monolith.py на пакетную структуру

Предложите архитектуру (какие модули, какие пакеты)

Реализуйте каркас пакета с правильными импортами

В каждом модуле используйте подходящий тип импорта (абсолютный/относительный)

Создайте __init__.py, который предоставляет простой интерфейс

Вариант 25. Кроссплатформенный конфигуратор
Тема проекта: Работа с путями на разных ОС

Структура пакета:

text
config_manager/
├── main.py
├── src/
│   └── config/
│       ├── __init__.py
│       ├── loaders/
│       │   ├── __init__.py
│       │   ├── json_loader.py
│       │   └── yaml_loader.py
│       ├── resolvers/
│       │   ├── __init__.py
│       │   └── path_resolver.py
│       └── models/
│           ├── __init__.py
│           └── config.py
└── configs/
Задание:

Создайте config.py с классом AppConfig

Создайте path_resolver.py с классом PathResolver, который:

Определяет ОС (platform.system())

Возвращает правильные пути для конфигов на Windows/Linux/macOS

Использует pathlib для всех операций

Создайте json_loader.py, который загружает конфиг с учётом ОС

В main.py используйте абсолютные импорты, чтобы загрузить и вывести конфиг

Демонстрируйте работу PathResolver с разными путями

Карточка студента (шаблон)
text
ПЗ 2.3. ВАРИАНТ № ___

Тема проекта: _________________________________

Структура пакета (нарисовать дерево папок):
_____________________________________________
_____________________________________________
_____________________________________________

Типы импортов, которые нужно использовать:
□ Абсолютные импорты в модуле: ______________
□ Относительные импорты в модуле: ___________
□ Импорт через __init__.py
□ Импорт с псевдонимом

Проверочный скрипт (что должен выводить main.py):
_____________________________________________
_____________________________________________

Дата выполнения: _____________
Таблица соответствия вариантов и проверочных действий
Вариант	Тема	Проверочное действие в main.py
0	Геометрия	Вычислить площадь круга и прямоугольника
1	Длина слов	Найти среднюю длину слова в тексте
2	Конвертер валют	Конвертировать 100 USD в RUB
3	Генератор паролей	Сгенерировать пароль длиной 12
4	Калории	Рассчитать калории для 150г продукта
5	Температура	Конвертировать 25°C в °F
6	To-Do	Добавить задачу и вывести список
7	Контакты	Сохранить и найти контакт
8	Бюджет	Посчитать сумму расходов за месяц
9	Библиотека	Найти книги автора
10	Рецепты	Масштабировать рецепт с 2 до 4 порций
11	Погода	Получить и отобразить погоду (мок-данные)
12	Цитаты	Загрузить и сохранить случайную цитату
13	QR-код	Сгенерировать QR для текста
14	Вакансии	Отфильтровать вакансии с зарплатой > 100k
15	Курсы валют	Конвертировать с кэшированием курса
16	Аутентификация	Зарегистрировать и аутентифицировать пользователя
17	Логирование	Записать сообщение в лог-файл
18	Отчёты	Сгенерировать PDF-отчёт из данных
19	Плагины	Загрузить и выполнить плагин по имени
20	Очередь задач	Добавить задачу и обработать воркером
21	Исправление ошибок	Исправить код так, чтобы он работал
22	Циклический импорт	Разрешить циклическую зависимость
23	Динамический импорт	Загрузить плагин из конфига
24	Монолит → пакет	Разбить на модули и импортировать
25	Кроссплатформенность	Загрузить конфиг с учётом ОС
Критерии оценки (для всех вариантов)
Баллы	Критерий
2 (неудовлетворительно)	Структура не соответствует требованию, импорты не работают
3 (удовлетворительно)	Создана базовая структура, но есть ошибки в импортах
4 (хорошо)	Все модули созданы, импорты работают, но структура неоптимальна
5 (отлично)	Идеальная структура, корректные импорты, работа с pathlib, есть __init__.py с __all__

# ПЗ 2.3. НУЛЕВОЙ ВАРИАНТ (ЭТАЛОН)
## Тема: Создание каркаса Python-пакета: модули и импорты
## Тип проекта: Геометрический калькулятор

---

## Назначение нулевого варианта

**Данный вариант выполняет ПРЕПОДАВАТЕЛЬ на занятии** как демонстрацию правильной последовательности действий.

Студенты наблюдают, записывают команды, задают вопросы. После демонстрации каждый студент получает свой индивидуальный вариант (1-25) и выполняет его самостоятельно.

**Что демонстрируется:**
- Создание структуры пакета из командной строки
- Создание файлов `__init__.py` (пустых и с содержимым)
- Написание модулей с разными типами импортов
- Исправление типичных ошибок импорта
- Работа с `pathlib` для кроссплатформенных путей

---

## Условие нулевого варианта

> Создайте пакет `geometry` для расчёта геометрических параметров фигур. Пакет должен содержать модули:
> - `models/shapes.py` — классы фигур (Circle, Rectangle, Triangle)
> - `calculators/area.py` — вычисление площади
> - `calculators/perimeter.py` — вычисление периметра
> - `utils/validators.py` — валидация входных данных
>
> В разных модулях должны использоваться **разные типы импортов**:
> - Относительные импорты (в `area.py`)
> - Абсолютные импорты (в `perimeter.py`)
> - Импорт через `__init__.py` (в `main.py`)
> - Импорт с псевдонимом (в `validators.py`)
>
> Главный модуль `main.py` должен демонстрировать работу всех функций и сохранять результаты в JSON с использованием `pathlib`.

---

## Эталонное выполнение

### Часть 1. Создание структуры пакета

#### Шаг 1.1. Создаём корневую папку

Откройте терминал в VS Code (`` Ctrl+` ``):

```bash
# Создаём папку проекта
mkdir geometry_package
cd geometry_package

# Показываем текущую папку
pwd   # или cd (Windows)
Шаг 1.2. Создаём структуру вложенных папок
macOS / Linux:

bash
# Создаём всю структуру одной командой
mkdir -p src/geometry/models
mkdir -p src/geometry/calculators
mkdir -p src/geometry/utils
mkdir -p tests
mkdir -p data

# Создаём все файлы __init__.py
touch src/__init__.py
touch src/geometry/__init__.py
touch src/geometry/models/__init__.py
touch src/geometry/calculators/__init__.py
touch src/geometry/utils/__init__.py
touch tests/__init__.py

# Создаём главный модуль
touch main.py

# Создаём служебные файлы
touch requirements.txt
touch README.md
touch .gitignore
Windows (PowerShell):

powershell
# Создаём структуру
New-Item -ItemType Directory -Force -Path src/geometry/models
New-Item -ItemType Directory -Force -Path src/geometry/calculators
New-Item -ItemType Directory -Force -Path src/geometry/utils
New-Item -ItemType Directory -Force -Path tests
New-Item -ItemType Directory -Force -Path data

# Создаём файлы __init__.py
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
Шаг 1.3. Проверяем структуру
bash
# macOS/Linux
tree

# Windows (если tree не работает, используйте dir /s)
dir /s
Ожидаемая структура:

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
│       │   ├── __init__.py
│       │   └── shapes.py      (создадим позже)
│       ├── calculators/
│       │   ├── __init__.py
│       │   ├── area.py        (создадим позже)
│       │   └── perimeter.py   (создадим позже)
│       └── utils/
│           ├── __init__.py
│           └── validators.py  (создадим позже)
├── tests/
│   └── __init__.py
└── data/
Часть 2. Создание модулей с разными типами импортов
Шаг 2.1. Создаём модуль моделей (shapes.py)
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
        """Диаметр круга."""
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
        """Является ли прямоугольник квадратом."""
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
        """Периметр треугольника."""
        return self.a + self.b + self.c
    
    def __str__(self) -> str:
        return f"Треугольник({self.a}, {self.b}, {self.c})"
Шаг 2.2. Создаём модуль площади (area.py) с ОТНОСИТЕЛЬНЫМ импортом
Создайте файл src/geometry/calculators/area.py:

python
"""
Модуль с калькулятором площади.
Использует ОТНОСИТЕЛЬНЫЙ импорт для доступа к моделям.
"""

# ОТНОСИТЕЛЬНЫЙ ИМПОРТ (работает только внутри пакета)
# Две точки .. означают "подняться на уровень выше"
from ..models.shapes import Circle, Rectangle, Triangle
import math
from typing import Union


class AreaCalculator:
    """Калькулятор площади фигур."""
    
    @staticmethod
    def circle_area(circle: Circle) -> float:
        """Вычисляет площадь круга: S = π * r²"""
        return math.pi * circle.radius ** 2
    
    @staticmethod
    def rectangle_area(rectangle: Rectangle) -> float:
        """Вычисляет площадь прямоугольника: S = width * height"""
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
ВНИМАНИЕ: Обратите внимание на строку from ..models.shapes import ... — это относительный импорт. Две точки означают переход в родительскую папку (geometry/), а затем в models/.

Шаг 2.3. Создаём модуль периметра (perimeter.py) с АБСОЛЮТНЫМ импортом
Создайте файл src/geometry/calculators/perimeter.py:

python
"""
Модуль с калькулятором периметра.
Использует АБСОЛЮТНЫЙ импорт для доступа к моделям.
"""

# АБСОЛЮТНЫЙ ИМПОРТ (указывает полный путь от корня проекта)
from src.geometry.models.shapes import Circle, Rectangle, Triangle
import math
from typing import Union


class PerimeterCalculator:
    """Калькулятор периметра фигур."""
    
    @staticmethod
    def circle_perimeter(circle: Circle) -> float:
        """Вычисляет длину окружности: P = 2 * π * r"""
        return 2 * math.pi * circle.radius
    
    @staticmethod
    def rectangle_perimeter(rectangle: Rectangle) -> float:
        """Вычисляет периметр прямоугольника: P = 2 * (w + h)"""
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
ВНИМАНИЕ: Обратите внимание на строку from src.geometry.models.shapes import ... — это абсолютный импорт. Путь указывается полностью от корня проекта (src/geometry/...).

Шаг 2.4. Создаём модуль валидаторов (validators.py) с импортом-ПСЕВДОНИМОМ
Создайте файл src/geometry/utils/validators.py:

python
"""
Модуль с утилитами для валидации.
Демонстрирует импорт с ПСЕВДОНИМОМ.
"""

from typing import Any, Optional
import json

# ИМПОРТ С ПСЕВДОНИМОМ (Path импортируется как P)
from pathlib import Path as P
from datetime import datetime as dt


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


def load_shapes_from_json(file_path: P) -> list:
    """Загружает фигуры из JSON-файла. Использует псевдоним P для Path."""
    if not file_path.exists():
        return []
    
    content = file_path.read_text(encoding='utf-8')
    data = json.loads(content)
    
    if isinstance(data, list):
        return data
    return []


def get_timestamp() -> str:
    """Возвращает текущую временную метку."""
    return dt.now().isoformat()
ВНИМАНИЕ: Обратите внимание на импорт from pathlib import Path as P — это импорт с псевдонимом. Теперь Path доступен как P.

Шаг 2.5. Настраиваем файл __init__.py для удобного импорта
Отредактируйте файл src/geometry/__init__.py:

python
"""
Пакет geometry — работа с геометрическими фигурами.
Позволяет вычислять площади и периметры.
"""

# Экспортируем основные классы и функции для удобного импорта
from .models.shapes import Circle, Rectangle, Triangle
from .calculators.area import AreaCalculator
from .calculators.perimeter import PerimeterCalculator
from .utils.validators import validate_positive, load_shapes_from_json

# Определяем, что импортируется при `from geometry import *`
__all__ = [
    'Circle',
    'Rectangle', 
    'Triangle',
    'AreaCalculator',
    'PerimeterCalculator',
    'validate_positive',
    'load_shapes_from_json'
]
ВНИМАНИЕ: В этом файле используются относительные импорты (с точкой). Файл __init__.py делает импорт всего пакета удобным.

Часть 3. Создание главного модуля с демонстрацией
Шаг 3.1. Создаём main.py
Создайте файл main.py в корне проекта:

python
#!/usr/bin/env python3
"""
Главный модуль демонстрирует различные способы импорта:
1. Импорт всего пакета (через __init__.py) — способ 1
2. Импорт конкретных классов (абсолютный) — способ 2
3. Импорт с псевдонимом — способ 3
4. Работа с pathlib для сохранения результатов
"""

from pathlib import Path
import json

# ============================================================
# СПОСОБ 1: импорт всего пакета (через __init__.py)
# ============================================================
# Благодаря __init__.py мы можем импортировать пакет целиком
import src.geometry as geom

print("=" * 50)
print("СПОСОБ 1: импорт всего пакета через __init__.py")
print("=" * 50)

# Создаём фигуры с помощью классов из пакета
circle = geom.Circle(radius=5)
rectangle = geom.Rectangle(width=4, height=6)
triangle = geom.Triangle(a=3, b=4, c=5)

# Вычисляем площади и периметры
print(f"\nКруг: {circle}")
print(f"  Площадь: {geom.AreaCalculator.circle_area(circle):.2f}")
print(f"  Длина окружности: {geom.PerimeterCalculator.circle_perimeter(circle):.2f}")

print(f"\nПрямоугольник: {rectangle}")
print(f"  Площадь: {geom.AreaCalculator.rectangle_area(rectangle):.2f}")
print(f"  Периметр: {geom.PerimeterCalculator.rectangle_perimeter(rectangle):.2f}")

print(f"\nТреугольник: {triangle}")
print(f"  Площадь: {geom.AreaCalculator.triangle_area(triangle):.2f}")
print(f"  Периметр: {geom.PerimeterCalculator.triangle_perimeter(triangle):.2f}")

# ============================================================
# СПОСОБ 2: импорт конкретных классов (абсолютный импорт)
# ============================================================
print("\n" + "=" * 50)
print("СПОСОБ 2: импорт конкретных классов (абсолютный)")
print("=" * 50)

from src.geometry.calculators.area import AreaCalculator
from src.geometry.calculators.perimeter import PerimeterCalculator
from src.geometry.models.shapes import Rectangle as Rect

# Используем импортированные классы
rect = Rect(width=10, height=5)
print(f"\nПрямоугольник 10×5:")
print(f"  Площадь: {AreaCalculator.rectangle_area(rect)}")
print(f"  Периметр: {PerimeterCalculator.rectangle_perimeter(rect)}")

# ============================================================
# СПОСОБ 3: импорт с псевдонимом
# ============================================================
print("\n" + "=" * 50)
print("СПОСОБ 3: импорт с псевдонимом")
print("=" * 50)

from src.geometry.utils.validators import validate_positive as vp
from src.geometry.utils.validators import get_timestamp as ts

try:
    vp(-5, "Радиус")
except ValueError as e:
    print(f"\nВалидация сработала: {e}")

print(f"\nТекущая временная метка: {ts()}")

# ============================================================
# РАБОТА С PATHLIB: сохранение результатов
# ============================================================
print("\n" + "=" * 50)
print("РАБОТА С PATHLIB (кроссплатформенные пути)")
print("=" * 50)

def save_results_to_json(results: list, filename: str = "results.json") -> Path:
    """
    Сохраняет результаты вычислений в JSON-файл.
    Использует pathlib для кроссплатформенной работы с путями.
    """
    # Определяем корневую папку проекта
    project_root = Path(__file__).parent
    
    # Создаём папку data (если её нет)
    data_dir = project_root / "data"
    data_dir.mkdir(exist_ok=True)
    
    # Формируем полный путь к файлу
    file_path = data_dir / filename
    
    # Подготавливаем данные для сохранения
    output = {
        "timestamp": ts(),
        "results": results
    }
    
    # Сохраняем в JSON
    file_path.write_text(
        json.dumps(output, indent=2, ensure_ascii=False), 
        encoding='utf-8'
    )
    
    print(f"\nРезультаты сохранены в: {file_path.absolute()}")
    return file_path

# Собираем результаты
results_data = [
    {
        "shape": "Круг",
        "radius": 5,
        "area": 78.54,
        "perimeter": 31.42
    },
    {
        "shape": "Прямоугольник",
        "width": 4,
        "height": 6,
        "area": 24.0,
        "perimeter": 20.0
    },
    {
        "shape": "Треугольник",
        "sides": [3, 4, 5],
        "area": 6.0,
        "perimeter": 12.0
    }
]

# Сохраняем
saved_path = save_results_to_json(results_data, "shapes_results.json")

# Демонстрация чтения файла с помощью pathlib
print(f"\nСодержимое сохранённого файла:")
print("-" * 40)
content = saved_path.read_text(encoding='utf-8')
print(content[:500] + "..." if len(content) > 500 else content)

print("\n" + "=" * 50)
print("✓ Все импорты работают корректно!")
print("=" * 50)
Часть 4. Запуск и проверка
Шаг 4.1. Запускаем проект
bash
# Убедитесь, что находитесь в корне проекта geometry_package
cd geometry_package

# Запускаем main.py
python main.py
Шаг 4.2. Ожидаемый вывод
text
==================================================
СПОСОБ 1: импорт всего пакета через __init__.py
==================================================

Круг: Круг(радиус=5)
  Площадь: 78.54
  Длина окружности: 31.42

Прямоугольник: Прямоугольник(4×6)
  Площадь: 24.00
  Периметр: 20.00

Треугольник: Треугольник(3, 4, 5)
  Площадь: 6.00
  Периметр: 12.00

==================================================
СПОСОБ 2: импорт конкретных классов (абсолютный)
==================================================

Прямоугольник 10×5:
  Площадь: 50.0
  Периметр: 30.0

==================================================
СПОСОБ 3: импорт с псевдонимом
==================================================

Валидация сработала: Радиус должно быть положительным, получено -5

Текущая временная метка: 2024-01-15T14:30:45.123456

==================================================
РАБОТА С PATHLIB (кроссплатформенные пути)
==================================================

Результаты сохранены в: /путь/к/geometry_package/data/shapes_results.json

Содержимое сохранённого файла:
----------------------------------------
{
  "timestamp": "2024-01-15T14:30:45.123456",
  "results": [
    {
      "shape": "Круг",
      "radius": 5,
      "area": 78.54,
      "perimeter": 31.42
    },
    {
      "shape": "Прямоугольник",
      "width": 4,
      "height": 6,
      "area": 24.0,
      "perimeter": 20.0
    },
    {
      "shape": "Треугольник",
      "sides": [3, 4, 5],
      "area": 6.0,
      "perimeter": 12.0
    }
  ]
}

==================================================
✓ Все импорты работают корректно!
==================================================
Часть 5. Проверка созданных файлов
Шаг 5.1. Проверяем содержимое папки data
bash
# macOS/Linux
cat data/shapes_results.json

# Windows
type data\shapes_results.json
Файл должен содержать валидный JSON с результатами.

Шаг 5.2. Проверяем структуру проекта
bash
# macOS/Linux
tree

# Windows
dir /s
Итоговая структура:

text
geometry_package/
├── main.py
├── requirements.txt (пустой)
├── README.md (пустой)
├── .gitignore
├── src/
│   ├── __init__.py (пустой)
│   └── geometry/
│       ├── __init__.py (содержит экспорты)
│       ├── models/
│       │   ├── __init__.py (пустой)
│       │   └── shapes.py
│       ├── calculators/
│       │   ├── __init__.py (пустой)
│       │   ├── area.py
│       │   └── perimeter.py
│       └── utils/
│           ├── __init__.py (пустой)
│           └── validators.py
├── tests/
│   └── __init__.py (пустой)
└── data/
    └── shapes_results.json
Что демонстрирует нулевой вариант
Демонстрируемый элемент	Где показан	Комментарий
Создание структуры пакета	Часть 1	Команды mkdir, touch
Относительные импорты	calculators/area.py	from ..models.shapes import ...
Абсолютные импорты	calculators/perimeter.py	from src.geometry.models.shapes import ...
Импорт через __init__.py	src/geometry/__init__.py + main.py	import src.geometry as geom
Импорт с псевдонимом	utils/validators.py	from pathlib import Path as P
__all__ в __init__.py	src/geometry/__init__.py	Ограничивает from package import *
Работа с pathlib	main.py, validators.py	Path(__file__).parent, mkdir(exist_ok=True)
Сохранение JSON	main.py	Кроссплатформенная запись файла
Обработка ошибок	validators.py, main.py	try/except при валидации
Контрольные вопросы для студентов (после демонстрации)
Почему в area.py используется относительный импорт, а в perimeter.py — абсолютный? В чём разница?

Что произойдёт, если удалить файл src/geometry/__init__.py? Проверьте.

Зачем нужен __all__ в __init__.py? Что произойдёт без него?

Почему в validators.py используется from pathlib import Path as P? Какая польза от псевдонима?

Как pathlib решает проблему кроссплатформенности? Приведите пример.

Что означает конструкция Path(__file__).parent в main.py?

Что студенты должны записать в тетрадь/отчёт
Структуру пакета (дерево папок)

Пример относительного импорта (строка кода)

Пример абсолютного импорта (строка кода)

Пример импорта с псевдонимом (строка кода)

Содержимое __init__.py (код)

Пример работы pathlib (создание папки, запись файла)

Распространённые ошибки (на что указать студентам)
Ошибка	Почему возникает	Как исправить
ImportError: attempted relative import with no known parent package	Относительный импорт используется в скрипте, запущенном напрямую	Использовать абсолютный импорт или запускать как модуль (python -m)
ModuleNotFoundError: No module named 'src'	Запуск не из корневой папки проекта	Перейти в папку geometry_package
AttributeError: module 'geometry' has no attribute 'Circle'	Не настроен __init__.py	Добавить экспорт в __init__.py
Файл data/shapes_results.json не создаётся	Папка data не существует	Использовать mkdir(parents=True, exist_ok=True)
Итог нулевого варианта
После демонстрации нулевого варианта студенты:

✅ Видели полный процесс создания пакета "с нуля"

✅ Поняли разницу между относительными и абсолютными импортами

✅ Узнали, как использовать __init__.py для удобного API пакета

✅ Освоили pathlib для кроссплатформенной работы с файлами

✅ Увидели, как сохранять результаты в JSON

Далее каждый студент получает свой индивидуальный вариант (1-25) и выполняет аналогичную работу самостоятельно.
