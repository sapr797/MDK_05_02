# ПЗ 2.4. 25 ВАРИАНТОВ ПРАКТИЧЕСКОЙ РАБОТЫ
## Тема: Рефакторинг кода: выделение модулей, организация импортов

---

## Нулевой вариант (эталонный) — Демонстрация преподавателя

**Назначение:** преподаватель выполняет этот вариант на занятии, показывая правильную последовательность действий при рефакторинге.

### Условие нулевого варианта

> Дан монолитный файл `library_monolith.py` (библиотечная система — 500+ строк). Выполните рефакторинг: разбейте код на модули и пакеты, организуйте правильные импорты (абсолютные и относительные), настройте `__init__.py`.

### Исходный монолит (демонстрационный)

```python
#!/usr/bin/env python3
"""
LIBRARY_MONOLITH.PY - Библиотечная система (ВСЁ В ОДНОМ ФАЙЛЕ)
Нужно РЕФАКТОРИНГОВАТЬ!
"""

import json
from datetime import datetime
from pathlib import Path
from typing import List, Optional

# ========== МОДЕЛИ ==========
class Book:
    def __init__(self, title: str, author: str, year: int, isbn: str):
        self.title = title
        self.author = author
        self.year = year
        self.isbn = isbn
        self.is_borrowed = False
        self.borrowed_by = None
        self.borrowed_date = None
    
    def to_dict(self) -> dict:
        return {'title': self.title, 'author': self.author, 'year': self.year, 
                'isbn': self.isbn, 'is_borrowed': self.is_borrowed,
                'borrowed_by': self.borrowed_by, 'borrowed_date': self.borrowed_date}
    
    @staticmethod
    def from_dict(data: dict):
        book = Book(data['title'], data['author'], data['year'], data['isbn'])
        book.is_borrowed = data.get('is_borrowed', False)
        book.borrowed_by = data.get('borrowed_by')
        book.borrowed_date = data.get('borrowed_date')
        return book

class Reader:
    def __init__(self, name: str, email: str, reader_id: str):
        self.name = name
        self.email = email
        self.reader_id = reader_id
        self.borrowed_books: List[str] = []
    
    def to_dict(self) -> dict:
        return {'name': self.name, 'email': self.email, 
                'reader_id': self.reader_id, 'borrowed_books': self.borrowed_books}
    
    @staticmethod
    def from_dict(data: dict):
        reader = Reader(data['name'], data['email'], data['reader_id'])
        reader.borrowed_books = data.get('borrowed_books', [])
        return reader

# ========== РЕПОЗИТОРИЙ ==========
class LibraryRepository:
    def __init__(self, data_dir: Path):
        self.data_dir = data_dir
        self.books_file = data_dir / 'books.json'
        self.readers_file = data_dir / 'readers.json'
        self._ensure_files_exist()
    
    def _ensure_files_exist(self):
        self.data_dir.mkdir(exist_ok=True)
        if not self.books_file.exists():
            self.books_file.write_text('[]', encoding='utf-8')
        if not self.readers_file.exists():
            self.readers_file.write_text('[]', encoding='utf-8')
    
    def get_all_books(self) -> List[Book]:
        return [Book.from_dict(b) for b in json.loads(self.books_file.read_text(encoding='utf-8'))]
    
    def save_book(self, book: Book):
        books = json.loads(self.books_file.read_text(encoding='utf-8'))
        books.append(book.to_dict())
        self.books_file.write_text(json.dumps(books, indent=2, ensure_ascii=False), encoding='utf-8')
    
    def find_book_by_isbn(self, isbn: str) -> Optional[Book]:
        for b in json.loads(self.books_file.read_text(encoding='utf-8')):
            if b['isbn'] == isbn:
                return Book.from_dict(b)
        return None
    
    def update_book(self, isbn: str, updated_book: Book):
        books = json.loads(self.books_file.read_text(encoding='utf-8'))
        for i, b in enumerate(books):
            if b['isbn'] == isbn:
                books[i] = updated_book.to_dict()
                break
        self.books_file.write_text(json.dumps(books, indent=2, ensure_ascii=False), encoding='utf-8')
    
    def get_all_readers(self) -> List[Reader]:
        return [Reader.from_dict(r) for r in json.loads(self.readers_file.read_text(encoding='utf-8'))]
    
    def save_reader(self, reader: Reader):
        readers = json.loads(self.readers_file.read_text(encoding='utf-8'))
        readers.append(reader.to_dict())
        self.readers_file.write_text(json.dumps(readers, indent=2, ensure_ascii=False), encoding='utf-8')
    
    def find_reader_by_id(self, reader_id: str) -> Optional[Reader]:
        for r in json.loads(self.readers_file.read_text(encoding='utf-8')):
            if r['reader_id'] == reader_id:
                return Reader.from_dict(r)
        return None
    
    def update_reader(self, reader_id: str, updated_reader: Reader):
        readers = json.loads(self.readers_file.read_text(encoding='utf-8'))
        for i, r in enumerate(readers):
            if r['reader_id'] == reader_id:
                readers[i] = updated_reader.to_dict()
                break
        self.readers_file.write_text(json.dumps(readers, indent=2, ensure_ascii=False), encoding='utf-8')

# ========== СЕРВИС ==========
class LibraryService:
    def __init__(self, repo: LibraryRepository):
        self.repo = repo
    
    def add_book(self, title: str, author: str, year: int, isbn: str) -> Book:
        if self.repo.find_book_by_isbn(isbn):
            raise ValueError(f"Книга с ISBN {isbn} уже существует")
        book = Book(title, author, year, isbn)
        self.repo.save_book(book)
        return book
    
    def borrow_book(self, isbn: str, reader_id: str) -> Book:
        book = self.repo.find_book_by_isbn(isbn)
        if not book:
            raise ValueError(f"Книга с ISBN {isbn} не найдена")
        if book.is_borrowed:
            raise ValueError(f"Книга '{book.title}' уже выдана")
        reader = self.repo.find_reader_by_id(reader_id)
        if not reader:
            raise ValueError(f"Читатель с ID {reader_id} не найден")
        
        book.is_borrowed = True
        book.borrowed_by = reader.name
        book.borrowed_date = datetime.now().isoformat()
        self.repo.update_book(isbn, book)
        
        reader.borrowed_books.append(isbn)
        self.repo.update_reader(reader_id, reader)
        return book

# ========== UI ==========
def main():
    data_dir = Path(__file__).parent / "library_data"
    repo = LibraryRepository(data_dir)
    service = LibraryService(repo)
    
    while True:
        print("\n=== БИБЛИОТЕКА ===")
        print("1. Добавить книгу")
        print("2. Выдать книгу")
        print("3. Выйти")
        choice = input("Выбор: ")
        
        if choice == '1':
            title = input("Название: ")
            author = input("Автор: ")
            year = int(input("Год: "))
            isbn = input("ISBN: ")
            try:
                book = service.add_book(title, author, year, isbn)
                print(f"✓ Книга добавлена: {book.title}")
            except ValueError as e:
                print(f"✗ {e}")
        elif choice == '2':
            isbn = input("ISBN книги: ")
            reader_id = input("ID читателя: ")
            try:
                book = service.borrow_book(isbn, reader_id)
                print(f"✓ Книга '{book.title}' выдана")
            except ValueError as e:
                print(f"✗ {e}")
        elif choice == '3':
            break

if __name__ == "__main__":
    main()
Целевая структура (демонстрационная)
text
library_system/
├── main.py
├── src/
│   ├── __init__.py
│   └── library/
│       ├── __init__.py
│       ├── models/
│       │   ├── __init__.py
│       │   ├── book.py
│       │   └── reader.py
│       ├── repositories/
│       │   ├── __init__.py
│       │   └── library_repo.py
│       ├── services/
│       │   ├── __init__.py
│       │   └── library_service.py
│       └── utils/
│           ├── __init__.py
│           └── validators.py
└── data/
Что демонстрирует преподаватель
Анализ монолита (выделение логических блоков)

Создание структуры папок

Разнесение кода по модулям

Настройка абсолютных импортов (в repositories, services, main.py)

Настройка относительных импортов (внутри пакета library)

Создание __init__.py с __all__

Запуск и проверка работоспособности

Базовый уровень (Варианты 1-10)
Вариант 1. Калькулятор (простые операции)
Исходный монолит: calculator.py — 4 операции (+, -, *, /), работа с историей, валидация ввода.

Задание: разбить на модули:

models/operation.py — класс Operation

calculators/basic.py — функции add, subtract, multiply, divide

services/history_service.py — сохранение/загрузка истории

ui/console.py — ввод/вывод

main.py — точка входа

Дополнительное требование: в basic.py использовать абсолютные импорты, в history_service.py — относительные.

Вариант 2. Генератор паролей
Исходный монолит: password_gen.py — генерация паролей разной сложности, проверка надёжности, сохранение в файл.

Задание: разбить на модули:

models/password.py — класс Password

generators/simple.py и generators/strong.py

validators/strength.py — проверка надёжности

exporters/file_saver.py — сохранение в файл

main.py

Дополнительное требование: в strength.py использовать импорт с псевдонимом для re (как regex).

Вариант 3. Конвертер температур
Исходный монолит: temp_converter.py — конвертация между C, F, K, история конвертаций.

Задание: разбить на модули:

models/temperature.py — класс Temperature

converters/celsius.py, converters/fahrenheit.py, converters/kelvin.py

services/conversion_service.py — основной сервис

storage/history.py — работа с историей

main.py

Дополнительное требование: в conversion_service.py использовать относительные импорты.

Вариант 4. Счётчик калорий
Исходный монолит: calorie_tracker.py — учёт продуктов, расчёт калорий, дневная норма.

Задание: разбить на модули:

models/food.py и models/meal.py

calculators/calorie_calc.py

services/daily_service.py

repositories/food_repo.py

main.py

Дополнительное требование: в food_repo.py использовать абсолютные импорты.

Вариант 5. Список дел (To-Do)
Исходный монолит: todo.py — добавление, удаление, отметка выполненных, сохранение в JSON.

Задание: разбить на модули:

models/task.py

repositories/task_repo.py

services/task_service.py

ui/display.py и ui/input_handler.py

main.py

Дополнительное требование: в task_service.py использовать относительный импорт из repositories.

Вариант 6. Контактная книга
Исходный монолит: contacts.py — добавление, поиск, редактирование, удаление контактов.

Задание: разбить на модули:

models/contact.py

repositories/contact_repo.py

services/contact_service.py

validators/email_validator.py и phone_validator.py

main.py

Дополнительное требование: в contact_service.py использовать абсолютные импорты из repositories и validators.

Вариант 7. Викторина (опросник)
Исходный монолит: quiz.py — вопросы, варианты ответов, подсчёт баллов.

Задание: разбить на модули:

models/question.py и models/quiz.py

loaders/question_loader.py — загрузка из JSON

services/quiz_service.py — логика опроса

reporters/score_reporter.py — вывод результатов

main.py

Дополнительное требование: в quiz_service.py использовать относительные импорты.

Вариант 8. Финансовый трекер
Исходный монолит: finance.py — доходы/расходы, категории, баланс.

Задание: разбить на модули:

models/transaction.py и models/category.py

repositories/transaction_repo.py

services/finance_service.py

analyzers/stats_analyzer.py

main.py

Дополнительное требование: в __init__.py пакета экспортировать только Transaction и FinanceService.

Вариант 9. Заметки с тегами
Исходный монолит: notes.py — создание заметок, теги, поиск по тегам.

Задание: разбить на модули:

models/note.py и models/tag.py

repositories/note_repo.py

services/note_service.py

search/tag_searcher.py

main.py

Дополнительное требование: в note_service.py использовать абсолютные импорты.

Вариант 10. Конвертер валют (фиксированные курсы)
Исходный монолит: currency.py — конвертация между USD, EUR, RUB, история.

Задание: разбить на модули:

models/currency.py и models/exchange_rate.py

converters/currency_converter.py

services/exchange_service.py

storage/history_storage.py

main.py

Дополнительное требование: в exchange_service.py использовать относительный импорт из converters.

Средний уровень (Варианты 11-20)
Вариант 11. Система бронирования отелей
Исходный монолит: hotel_booking.py — отели, номера, бронирование, проверка доступности.

Задание: разбить на модули:

models/hotel.py, models/room.py, models/booking.py

repositories/hotel_repo.py, room_repo.py

services/booking_service.py

validators/date_validator.py

exporters/invoice_exporter.py

main.py

Дополнительное требование: добавить utils/logger.py для логирования действий.

Вариант 12. Планировщик задач (с приоритетами)
Исходный монолит: scheduler.py — задачи, приоритеты, дедлайны, уведомления.

Задание: разбить на модули:

models/task.py, models/priority.py

repositories/task_repo.py

services/scheduler_service.py

notifiers/email_notifier.py

sorters/priority_sorter.py

main.py

Дополнительное требование: в scheduler_service.py использовать абсолютные импорты для notifiers и относительные для repositories.

Вариант 13. Менеджер паролей (с шифрованием)
Исходный монолит: password_manager.py — сохранение паролей, шифрование, мастер-пароль.

Задание: разбить на модули:

models/account.py, models/encrypted_entry.py

crypto/aes_encryptor.py, crypto/hasher.py

repositories/vault_repo.py

services/vault_service.py

cli/commands.py

main.py

Дополнительное требование: создать __init__.py, который экспортирует VaultService и Account.

Вариант 14. Система голосования
Исходный монолит: voting.py — кандидаты, голоса, подсчёт результатов.

Задание: разбить на модули:

models/candidate.py, models/vote.py, models/voter.py

repositories/vote_repo.py

services/voting_service.py

analyzers/result_analyzer.py

reporters/result_reporter.py

main.py

Дополнительное требование: в voting_service.py использовать относительные импорты для всех внутренних модулей.

Вариант 15. Блог с комментариями
Исходный монолит: blog.py — посты, комментарии, авторы, даты.

Задание: разбить на модули:

models/post.py, models/comment.py, models/author.py

repositories/post_repo.py, comment_repo.py

services/blog_service.py

formatters/date_formatter.py, text_formatter.py

main.py

Дополнительное требование: добавить utils/markdown.py для форматирования текста (импорт с псевдонимом).

Вариант 16. Магазин (корзина, товары, заказы)
Исходный монолит: store.py — товары, корзина, заказы, расчёт доставки.

Задание: разбить на модули:

models/product.py, models/cart.py, models/order.py

repositories/product_repo.py, order_repo.py

services/cart_service.py, order_service.py

calculators/delivery_calculator.py

main.py

Дополнительное требование: создать пакет src/store с вложенными пакетами models, services, repositories.

Вариант 17. Система управления библиотекой (расширенная)
Исходный монолит: library_advanced.py — книги, читатели, авторы, жанры, отзывы.

Задание: разбить на модули:

models/book.py, reader.py, author.py, genre.py, review.py

repositories/book_repo.py, reader_repo.py, review_repo.py

services/library_service.py, review_service.py

search/search_engine.py

main.py

Дополнительное требование: в library_service.py использовать абсолютные импорты, в review_service.py — относительные.

Вариант 18. Финансовый отчёт (анализ CSV)
Исходный монолит: finance_report.py — чтение CSV, обработка, группировка, экспорт в JSON.

Задание: разбить на модули:

models/transaction.py, models/category.py

parsers/csv_parser.py

analyzers/group_analyzer.py, sum_analyzer.py

exporters/json_exporter.py, excel_exporter.py

services/report_service.py

main.py

Дополнительное требование: в report_service.py использовать динамический импорт для экспортёров.

Вариант 19. Telegram-бот (эхо-бот с логированием)
Исходный монолит: telegram_bot.py — обработка сообщений, логирование, команды.

Задание: разбить на модули:

handlers/message_handler.py, command_handler.py

services/bot_service.py

loggers/file_logger.py, console_logger.py

config/settings.py (с pathlib)

main.py

Дополнительное требование: в settings.py использовать pathlib для путей к логам (кроссплатформенно).

Вариант 20. Система тестирования (с вопросами и ответами)
Исходный монолит: testing_system.py — тесты, вопросы, варианты ответов, подсчёт баллов, сохранение результатов.

Задание: разбить на модули:

models/test.py, question.py, answer.py, result.py

repositories/test_repo.py, result_repo.py

services/testing_service.py, scoring_service.py

ui/test_ui.py, result_ui.py

main.py

Дополнительное требование: в __init__.py пакета src/testing экспортировать только TestingService и Test.

Сложный уровень (Варианты 21-25)
Вариант 21. REST API клиент (с кэшированием)
Исходный монолит: api_client.py — запросы к API, кэширование, обработка ошибок, ретраи.

Задание: разбить на модули:

models/request.py, response.py, cache_entry.py

clients/http_client.py

cache/memory_cache.py, file_cache.py

handlers/error_handler.py, retry_handler.py

services/api_service.py

main.py

Дополнительное требование: в api_service.py реализовать циклический импорт между clients и handlers и разрешить его.

Вариант 22. Веб-скрапер с пайплайном обработки
Исходный монолит: scraper.py — загрузка страниц, парсинг, очистка данных, сохранение в БД.

Задание: разбить на модули:

models/raw_page.py, parsed_data.py, clean_data.py

downloaders/page_downloader.py

parsers/html_parser.py, json_parser.py

cleaners/text_cleaner.py, html_cleaner.py

pipeline/pipeline.py, stages.py

storage/db_storage.py, file_storage.py

main.py

Дополнительное требование: создать конфигурационный файл config.yaml, загружать его через pathlib и yaml.

Вариант 23. Система мониторинга серверов
Исходный монолит: monitoring.py — проверка серверов (ping, HTTP), алерты, логирование в БД.

Задание: разбить на модули:

models/server.py, check_result.py, alert.py

checkers/ping_checker.py, http_checker.py, port_checker.py

alerters/email_alerter.py, console_alerter.py

storage/db_storage.py (с использованием SQLAlchemy)

schedulers/scheduler.py (с schedule или apscheduler)

config/config_loader.py

main.py

Дополнительное требование: в config_loader.py использовать динамический импорт для загрузки нужных чекеров по имени из конфига.

Вариант 24. Обработчик логов (Log Analyzer)
Исходный монолит: log_analyzer.py — чтение лог-файлов, парсинг форматов, агрегация, отчёт.

Задание: разбить на модули:

models/log_entry.py, log_level.py, aggregated_data.py

readers/file_reader.py, stdin_reader.py, socket_reader.py

parsers/apache_parser.py, nginx_parser.py, json_parser.py, regex_parser.py

analyzers/frequency_analyzer.py, timeline_analyzer.py, error_analyzer.py

exporters/html_exporter.py, json_exporter.py, csv_exporter.py

pipeline/log_pipeline.py

main.py

Дополнительное требование: реализовать паттерн Стратегия для парсеров через абстрактный класс и динамическую загрузку.

Вариант 25. Фреймворк для ETL-процессов
Исходный монолит: etl_framework.py — Extract (CSV/JSON/API), Transform (фильтрация, агрегация), Load (в БД).

Задание: разбить на модули:

models/row.py, schema.py, dataset.py

extractors/csv_extractor.py, json_extractor.py, api_extractor.py

transformers/filter_transformer.py, aggregate_transformer.py, join_transformer.py, map_transformer.py

loaders/db_loader.py, file_loader.py, api_loader.py

engine/etl_engine.py, pipeline.py

config/pipeline_config.py (поддержка YAML/JSON)

main.py

Дополнительное требование:

Создать абстрактные классы для всех компонентов (Extractor, Transformer, Loader)

Реализовать фабрику для создания компонентов по имени из конфига

В etl_engine.py использовать относительные импорты с поднятием на 2 уровня (from ...extractors import ...)

Карточка студента (шаблон)
text
ПЗ 2.4. РЕФАКТОРИНГ КОДА

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

Название проекта: _________________________________

Исходный монолит: _________________________________

Требуемая структура (схематично):
___________________________________________________
___________________________________________________
___________________________________________________

Типы импортов, которые нужно использовать:
□ Абсолютные импорты — где: ______________________
□ Относительные импорты — где: ___________________
□ Импорт с псевдонимом — где: ____________________
□ Импорт через __init__.py

Дополнительные требования: ________________________
___________________________________________________

Дата выполнения: _____________
Таблица соответствия вариантов и типов импортов
Вариант	Абсолютные импорты	Относительные импорты	Импорт с псевдонимом	Через __init__.py	Доп. сложность
0	repos, main	services	validators	да	pathlib
1	basic.py	history_service.py	нет	да	нет
2	нет	generators	strength.py (re)	да	нет
3	нет	conversion_service.py	нет	да	нет
4	food_repo.py	нет	нет	да	нет
5	нет	task_service.py	нет	да	нет
6	contact_service.py	нет	нет	да	нет
7	нет	quiz_service.py	нет	да	нет
8	нет	через init	нет	да	да
9	note_service.py	нет	нет	да	нет
10	нет	exchange_service.py	нет	да	нет
11	invoice_exporter.py	booking_service.py	нет	да	logger.py
12	notifiers	repositories	нет	да	нет
13	vault_service.py	crypto	нет	да	шифрование
14	нет	voting_service.py	нет	да	нет
15	нет	blog_service.py	markdown.py	да	нет
16	cart_service.py	order_service.py	нет	да	вложенные пакеты
17	library_service.py	review_service.py	нет	да	search_engine
18	report_service.py	analyzers	нет	да	динамический импорт
19	settings.py	handlers, services	нет	да	pathlib
20	нет	testing_service.py	нет	да	нет
21	api_service.py	clients, handlers	нет	да	циклический импорт
22	pipeline	stages	нет	да	config.yaml
23	checkers	alerters	нет	да	динамический импорт
24	pipeline	parsers, analyzers	regex_parser.py	да	паттерн Стратегия
25	engine	extractors, transformers	нет	да	абстрактные классы + фабрика
Критерии оценки (для всех вариантов)
Баллы	Критерий
2 (неудовлетворительно)	Рефакторинг не выполнен или программа не работает
3 (удовлетворительно)	Создана структура, но импорты работают не все, программа запускается с ошибками
4 (хорошо)	Все модули созданы, импорты работают, программа запускается без ошибок
5 (отлично)	Полностью рабочий проект, корректные импорты, есть __init__.py с __all__, код соответствует PEP8, выполнены дополнительные требования
Общие требования ко всем вариантам
Структура: проект должен быть организован как пакет с папкой src/.

Импорты: использовать разные типы импортов согласно таблице выше.

__init__.py: в каждом пакете должен быть файл __init__.py (может быть пустым, кроме главного).

__all__: в src/<project>/__init__.py должен быть определён __all__.

pathlib: все пути к файлам должны быть кроссплатформенными (использовать pathlib).

Документация: каждый модуль должен содержать docstring с описанием.

Сохранение функциональности: после рефакторинга программа должна работать так же, как исходный монолит.

Тестирование: приложить скриншот успешного выполнения.

Шпаргалка по рефакторингу для студентов
python
# Принципы рефакторинга:

# 1. Один модуль = одна ответственность
# ПЛОХО: models.py содержит и Book, и Reader, и Author
# ХОРОШО: models/book.py, models/reader.py, models/author.py

# 2. Разделение на слои:
# - models/ — только данные (dataclasses, базовые классы)
# - repositories/ — работа с хранилищем (файлы, БД)
# - services/ — бизнес-логика
# - ui/ или api/ — интерфейсы
# - utils/ — вспомогательные функции

# 3. Правила импортов:
# - Модели не импортируют ничего (или только стандартную библиотеку)
# - Репозитории импортируют модели (абсолютно)
# - Сервисы импортируют репозитории и модели (относительно внутри пакета)
# - UI импортирует сервисы (абсолютно)

# 4. Абсолютные импорты (в main.py, ui/, репозиториях):
from src.library.models.book import Book
from src.library.services.library_service import LibraryService

# 5. Относительные импорты (внутри пакета library):
from ..models.book import Book
from .library_repo import LibraryRepository

# 6. __init__.py для удобного API:
from .models.book import Book
from .services.library_service import LibraryService
__all__ = ['Book', 'LibraryService']
