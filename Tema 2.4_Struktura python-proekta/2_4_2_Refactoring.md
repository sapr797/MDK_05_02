# ПЗ 2.4. Рефакторинг кода: выделение модулей, организация импортов

**Тема:** Структура Python-проекта, рефакторинг, модули, пакеты, импорты

**Цель работы:**  
Научиться преобразовывать "плохой" монолитный код в правильно организованный Python-пакет, выделять модули по принципу единственной ответственности, настраивать абсолютные и относительные импорты.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Git (опционально)

---

## Теоретическая справка: что такое рефакторинг

**Рефакторинг** — это процесс изменения внутренней структуры кода без изменения его внешнего поведения.

**Признаки "плохого" кода, требующего рефакторинга:**

| Признак | Описание | Как исправить |
|---------|----------|---------------|
| Длинный метод | Функция на 100+ строк | Разбить на меньшие функции |
| Божественный объект | Класс, который делает всё | Выделить специализированные классы |
| Дублирование кода | Одинаковые фрагменты в разных местах | Вынести в общую функцию |
| Спагетти-код | Запутанные связи между частями | Разделить на модули по ответственности |
| Один файл на всё | Весь проект в одном `.py` | Разбить на пакет и модули |

**В этой работе вы превратите монолит в правильно организованный пакет.**

---

## Часть 1. Анализ "плохого" кода (15 минут)

### Исходный файл: `monolith.py`

Вам дан файл `monolith.py` — это программа для управления библиотекой, написанная в одном файле. Проанализируйте его структуру.

```python
#!/usr/bin/env python3
"""
MONOLITH.PY - Библиотечная система (ВСЁ В ОДНОМ ФАЙЛЕ)
Этот файл содержит всё: модели, репозиторий, сервисы, UI.
Нужно РЕФАКТОРИНГОВАТЬ!
"""

import json
import os
from datetime import datetime
from pathlib import Path
from typing import List, Dict, Optional

# ============================================================
# МОДЕЛИ (данные)
# ============================================================

class Book:
    """Модель книги."""
    def __init__(self, title: str, author: str, year: int, isbn: str):
        self.title = title
        self.author = author
        self.year = year
        self.isbn = isbn
        self.is_borrowed = False
        self.borrowed_by = None
        self.borrowed_date = None
    
    def to_dict(self) -> dict:
        return {
            'title': self.title,
            'author': self.author,
            'year': self.year,
            'isbn': self.isbn,
            'is_borrowed': self.is_borrowed,
            'borrowed_by': self.borrowed_by,
            'borrowed_date': self.borrowed_date
        }
    
    @staticmethod
    def from_dict(data: dict):
        book = Book(data['title'], data['author'], data['year'], data['isbn'])
        book.is_borrowed = data.get('is_borrowed', False)
        book.borrowed_by = data.get('borrowed_by')
        book.borrowed_date = data.get('borrowed_date')
        return book
    
    def __str__(self):
        status = "✓ В наличии" if not self.is_borrowed else f"✗ Выдана: {self.borrowed_by}"
        return f"{self.title} — {self.author} ({self.year}) [{self.isbn}] {status}"


class Reader:
    """Модель читателя."""
    def __init__(self, name: str, email: str, reader_id: str):
        self.name = name
        self.email = email
        self.reader_id = reader_id
        self.borrowed_books: List[str] = []  # список ISBN
    
    def to_dict(self) -> dict:
        return {
            'name': self.name,
            'email': self.email,
            'reader_id': self.reader_id,
            'borrowed_books': self.borrowed_books
        }
    
    @staticmethod
    def from_dict(data: dict):
        reader = Reader(data['name'], data['email'], data['reader_id'])
        reader.borrowed_books = data.get('borrowed_books', [])
        return reader
    
    def __str__(self):
        return f"{self.name} ({self.email}) ID: {self.reader_id}, книг на руках: {len(self.borrowed_books)}"


# ============================================================
# РЕПОЗИТОРИЙ (работа с файлами)
# ============================================================

class LibraryRepository:
    """Репозиторий для работы с JSON-файлами."""
    
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
    
    def _load_books(self) -> List[dict]:
        return json.loads(self.books_file.read_text(encoding='utf-8'))
    
    def _save_books(self, books: List[dict]):
        self.books_file.write_text(json.dumps(books, indent=2, ensure_ascii=False), encoding='utf-8')
    
    def _load_readers(self) -> List[dict]:
        return json.loads(self.readers_file.read_text(encoding='utf-8'))
    
    def _save_readers(self, readers: List[dict]):
        self.readers_file.write_text(json.dumps(readers, indent=2, ensure_ascii=False), encoding='utf-8')
    
    def get_all_books(self) -> List[Book]:
        books_data = self._load_books()
        return [Book.from_dict(b) for b in books_data]
    
    def save_book(self, book: Book):
        books = self._load_books()
        books.append(book.to_dict())
        self._save_books(books)
    
    def update_book(self, isbn: str, updated_book: Book):
        books = self._load_books()
        for i, b in enumerate(books):
            if b['isbn'] == isbn:
                books[i] = updated_book.to_dict()
                break
        self._save_books(books)
    
    def find_book_by_isbn(self, isbn: str) -> Optional[Book]:
        books = self._load_books()
        for b in books:
            if b['isbn'] == isbn:
                return Book.from_dict(b)
        return None
    
    def get_all_readers(self) -> List[Reader]:
        readers_data = self._load_readers()
        return [Reader.from_dict(r) for r in readers_data]
    
    def save_reader(self, reader: Reader):
        readers = self._load_readers()
        readers.append(reader.to_dict())
        self._save_readers(readers)
    
    def update_reader(self, reader_id: str, updated_reader: Reader):
        readers = self._load_readers()
        for i, r in enumerate(readers):
            if r['reader_id'] == reader_id:
                readers[i] = updated_reader.to_dict()
                break
        self._save_readers(readers)
    
    def find_reader_by_id(self, reader_id: str) -> Optional[Reader]:
        readers = self._load_readers()
        for r in readers:
            if r['reader_id'] == reader_id:
                return Reader.from_dict(r)
        return None


# ============================================================
# БИЗНЕС-ЛОГИКА (СЕРВИСЫ)
# ============================================================

class LibraryService:
    """Сервис с бизнес-логикой библиотеки."""
    
    def __init__(self, repository: LibraryRepository):
        self.repo = repository
    
    def add_book(self, title: str, author: str, year: int, isbn: str) -> Book:
        """Добавляет новую книгу."""
        # Проверка: книга с таким ISBN уже существует?
        existing = self.repo.find_book_by_isbn(isbn)
        if existing:
            raise ValueError(f"Книга с ISBN {isbn} уже существует")
        
        book = Book(title, author, year, isbn)
        self.repo.save_book(book)
        return book
    
    def add_reader(self, name: str, email: str, reader_id: str) -> Reader:
        """Добавляет нового читателя."""
        existing = self.repo.find_reader_by_id(reader_id)
        if existing:
            raise ValueError(f"Читатель с ID {reader_id} уже существует")
        
        reader = Reader(name, email, reader_id)
        self.repo.save_reader(reader)
        return reader
    
    def borrow_book(self, isbn: str, reader_id: str) -> Book:
        """Выдаёт книгу читателю."""
        book = self.repo.find_book_by_isbn(isbn)
        if not book:
            raise ValueError(f"Книга с ISBN {isbn} не найдена")
        
        if book.is_borrowed:
            raise ValueError(f"Книга '{book.title}' уже выдана")
        
        reader = self.repo.find_reader_by_id(reader_id)
        if not reader:
            raise ValueError(f"Читатель с ID {reader_id} не найден")
        
        # Обновляем книгу
        book.is_borrowed = True
        book.borrowed_by = reader.name
        book.borrowed_date = datetime.now().isoformat()
        self.repo.update_book(isbn, book)
        
        # Обновляем читателя
        reader.borrowed_books.append(isbn)
        self.repo.update_reader(reader_id, reader)
        
        return book
    
    def return_book(self, isbn: str) -> Book:
        """Возвращает книгу обратно в библиотеку."""
        book = self.repo.find_book_by_isbn(isbn)
        if not book:
            raise ValueError(f"Книга с ISBN {isbn} не найдена")
        
        if not book.is_borrowed:
            raise ValueError(f"Книга '{book.title}' не была выдана")
        
        # Находим читателя, который брал книгу
        readers = self.repo.get_all_readers()
        reader = next((r for r in readers if isbn in r.borrowed_books), None)
        
        # Обновляем книгу
        book.is_borrowed = False
        book.borrowed_by = None
        book.borrowed_date = None
        self.repo.update_book(isbn, book)
        
        # Обновляем читателя
        if reader:
            reader.borrowed_books.remove(isbn)
            self.repo.update_reader(reader.reader_id, reader)
        
        return book
    
    def search_books(self, query: str) -> List[Book]:
        """Ищет книги по названию или автору."""
        all_books = self.repo.get_all_books()
        query_lower = query.lower()
        return [
            book for book in all_books
            if query_lower in book.title.lower() or query_lower in book.author.lower()
        ]
    
    def get_available_books(self) -> List[Book]:
        """Возвращает список доступных книг."""
        all_books = self.repo.get_all_books()
        return [book for book in all_books if not book.is_borrowed]
    
    def get_borrowed_books(self) -> List[Book]:
        """Возвращает список выданных книг."""
        all_books = self.repo.get_all_books()
        return [book for book in all_books if book.is_borrowed]
    
    def get_reader_books(self, reader_id: str) -> List[Book]:
        """Возвращает список книг, которые взял читатель."""
        reader = self.repo.find_reader_by_id(reader_id)
        if not reader:
            return []
        
        all_books = self.repo.get_all_books()
        return [book for book in all_books if book.isbn in reader.borrowed_books]


# ============================================================
# ПОЛЬЗОВАТЕЛЬСКИЙ ИНТЕРФЕЙС (UI)
# ============================================================

def print_header(title: str):
    """Печатает заголовок."""
    print("\n" + "=" * 50)
    print(f"  {title}")
    print("=" * 50)


def print_books(books: List[Book]):
    """Печатает список книг."""
    if not books:
        print("  Нет книг")
        return
    for book in books:
        print(f"  • {book}")


def print_readers(readers: List[Reader]):
    """Печатает список читателей."""
    if not readers:
        print("  Нет читателей")
        return
    for reader in readers:
        print(f"  • {reader}")


def main():
    """Главная функция программы."""
    # Инициализация
    data_dir = Path(__file__).parent / "library_data"
    repo = LibraryRepository(data_dir)
    service = LibraryService(repo)
    
    while True:
        print_header("БИБЛИОТЕЧНАЯ СИСТЕМА")
        print("1. Добавить книгу")
        print("2. Добавить читателя")
        print("3. Выдать книгу")
        print("4. Вернуть книгу")
        print("5. Поиск книг")
        print("6. Список всех книг")
        print("7. Доступные книги")
        print("8. Выданные книги")
        print("9. Книги читателя")
        print("10. Выйти")
        
        choice = input("\nВыберите действие: ")
        
        if choice == '1':
            print_header("ДОБАВЛЕНИЕ КНИГИ")
            title = input("Название: ")
            author = input("Автор: ")
            year = int(input("Год: "))
            isbn = input("ISBN: ")
            try:
                book = service.add_book(title, author, year, isbn)
                print(f"✓ Книга добавлена: {book.title}")
            except ValueError as e:
                print(f"✗ Ошибка: {e}")
        
        elif choice == '2':
            print_header("ДОБАВЛЕНИЕ ЧИТАТЕЛЯ")
            name = input("Имя: ")
            email = input("Email: ")
            reader_id = input("ID читателя: ")
            try:
                reader = service.add_reader(name, email, reader_id)
                print(f"✓ Читатель добавлен: {reader.name}")
            except ValueError as e:
                print(f"✗ Ошибка: {e}")
        
        elif choice == '3':
            print_header("ВЫДАЧА КНИГИ")
            isbn = input("ISBN книги: ")
            reader_id = input("ID читателя: ")
            try:
                book = service.borrow_book(isbn, reader_id)
                print(f"✓ Книга '{book.title}' выдана")
            except ValueError as e:
                print(f"✗ Ошибка: {e}")
        
        elif choice == '4':
            print_header("ВОЗВРАТ КНИГИ")
            isbn = input("ISBN книги: ")
            try:
                book = service.return_book(isbn)
                print(f"✓ Книга '{book.title}' возвращена")
            except ValueError as e:
                print(f"✗ Ошибка: {e}")
        
        elif choice == '5':
            print_header("ПОИСК КНИГ")
            query = input("Поисковый запрос: ")
            books = service.search_books(query)
            print_books(books)
        
        elif choice == '6':
            print_header("ВСЕ КНИГИ")
            books = service.repo.get_all_books()
            print_books(books)
        
        elif choice == '7':
            print_header("ДОСТУПНЫЕ КНИГИ")
            books = service.get_available_books()
            print_books(books)
        
        elif choice == '8':
            print_header("ВЫДАННЫЕ КНИГИ")
            books = service.get_borrowed_books()
            print_books(books)
        
        elif choice == '9':
            print_header("КНИГИ ЧИТАТЕЛЯ")
            reader_id = input("ID читателя: ")
            books = service.get_reader_books(reader_id)
            print_books(books)
        
        elif choice == '10':
            print("До свидания!")
            break
        
        else:
            print("Неверный выбор, попробуйте снова")


if __name__ == "__main__":
    main()
Задание 1.1. Анализ монолита (ответьте письменно)
Ответьте на следующие вопросы (письменно в отчёте):

Сколько строк в файле monolith.py? (примерно)

Какие логические компоненты можно выделить в этом файле?
Подсказка: модели, репозиторий, сервисы, UI

Какие проблемы вы видите в текущей организации кода?
Перечислите минимум 3 проблемы

Как вы предложите разбить этот код на модули и пакеты?
Нарисуйте схематично структуру папок

Какие типы импортов (абсолютные/относительные) где стоит использовать?

Часть 2. Планирование рефакторинга (15 минут)
Задание 2.1. Определите целевую структуру
На основе анализа предложите структуру пакета.

Ожидаемая структура (рекомендуемая):

text
library_system/
├── main.py                      # точка входа (только запуск UI)
├── requirements.txt
├── README.md
├── .gitignore
│
├── src/
│   ├── __init__.py
│   │
│   ├── library/                 # основной пакет
│   │   ├── __init__.py
│   │   │
│   │   ├── models/              # модели данных
│   │   │   ├── __init__.py
│   │   │   ├── book.py
│   │   │   └── reader.py
│   │   │
│   │   ├── repositories/        # работа с хранилищем
│   │   │   ├── __init__.py
│   │   │   └── library_repo.py
│   │   │
│   │   ├── services/            # бизнес-логика
│   │   │   ├── __init__.py
│   │   │   └── library_service.py
│   │   │
│   │   └── utils/               # вспомогательные функции
│   │       ├── __init__.py
│   │       └── validators.py
│   │
│   └── ui/                      # пользовательский интерфейс
│       ├── __init__.py
│       ├── menu.py
│       └── display.py
│
└── data/                        # данные (JSON-файлы)
    ├── books.json
    └── readers.json
Задание 2.2. Заполните таблицу распределения ответственности
Компонент	Какие функции/классы войдут	Какой тип импорта использовать
models/book.py	Book	Без импортов (независим)
models/reader.py	Reader	Без импортов
repositories/library_repo.py	LibraryRepository	Абсолютный (из models)
services/library_service.py	LibraryService	Относительный или абсолютный
ui/menu.py	main_menu()	Абсолютный (из services)
ui/display.py	print_books(), print_readers()	Абсолютный (из models)
utils/validators.py	функции валидации	Без импортов
main.py	запуск	Абсолютный (из ui)
Часть 3. Выполнение рефакторинга (45 минут)
Задание 3.1. Создайте структуру папок
Выполните команды в терминале:

macOS/Linux:

bash
# Создаём проект
mkdir library_system
cd library_system

# Создаём структуру
mkdir -p src/library/models
mkdir -p src/library/repositories
mkdir -p src/library/services
mkdir -p src/library/utils
mkdir -p src/ui
mkdir -p data

# Создаём файлы __init__.py
touch src/__init__.py
touch src/library/__init__.py
touch src/library/models/__init__.py
touch src/library/repositories/__init__.py
touch src/library/services/__init__.py
touch src/library/utils/__init__.py
touch src/ui/__init__.py

# Создаём основные файлы
touch src/library/models/book.py
touch src/library/models/reader.py
touch src/library/repositories/library_repo.py
touch src/library/services/library_service.py
touch src/library/utils/validators.py
touch src/ui/menu.py
touch src/ui/display.py
touch main.py

# Создаём служебные файлы
touch requirements.txt README.md .gitignore
Windows (PowerShell):

powershell
# Создаём проект
mkdir library_system
cd library_system

# Создаём структуру
New-Item -ItemType Directory -Force -Path src/library/models
New-Item -ItemType Directory -Force -Path src/library/repositories
New-Item -ItemType Directory -Force -Path src/library/services
New-Item -ItemType Directory -Force -Path src/library/utils
New-Item -ItemType Directory -Force -Path src/ui
New-Item -ItemType Directory -Force -Path data

# Создаём файлы __init__.py
New-Item -ItemType File -Force -Path src/__init__.py
New-Item -ItemType File -Force -Path src/library/__init__.py
New-Item -ItemType File -Force -Path src/library/models/__init__.py
New-Item -ItemType File -Force -Path src/library/repositories/__init__.py
New-Item -ItemType File -Force -Path src/library/services/__init__.py
New-Item -ItemType File -Force -Path src/library/utils/__init__.py
New-Item -ItemType File -Force -Path src/ui/__init__.py

# Создаём основные файлы
New-Item -ItemType File -Force -Path src/library/models/book.py
New-Item -ItemType File -Force -Path src/library/models/reader.py
New-Item -ItemType File -Force -Path src/library/repositories/library_repo.py
New-Item -ItemType File -Force -Path src/library/services/library_service.py
New-Item -ItemType File -Force -Path src/library/utils/validators.py
New-Item -ItemType File -Force -Path src/ui/menu.py
New-Item -ItemType File -Force -Path src/ui/display.py
New-Item -ItemType File -Force -Path main.py

# Создаём служебные файлы
New-Item -ItemType File -Force -Path requirements.txt, README.md, .gitignore
Задание 3.2. Создайте модуль моделей
Файл src/library/models/book.py:

python
"""
Модуль модели книги.
Содержит класс Book и методы для работы с ним.
"""

from datetime import datetime
from typing import Optional


class Book:
    """Модель книги в библиотеке."""
    
    def __init__(self, title: str, author: str, year: int, isbn: str):
        self.title = title
        self.author = author
        self.year = year
        self.isbn = isbn
        self.is_borrowed = False
        self.borrowed_by: Optional[str] = None
        self.borrowed_date: Optional[str] = None
    
    def to_dict(self) -> dict:
        """Преобразует книгу в словарь для JSON."""
        return {
            'title': self.title,
            'author': self.author,
            'year': self.year,
            'isbn': self.isbn,
            'is_borrowed': self.is_borrowed,
            'borrowed_by': self.borrowed_by,
            'borrowed_date': self.borrowed_date
        }
    
    @staticmethod
    def from_dict(data: dict) -> 'Book':
        """Создаёт книгу из словаря."""
        book = Book(data['title'], data['author'], data['year'], data['isbn'])
        book.is_borrowed = data.get('is_borrowed', False)
        book.borrowed_by = data.get('borrowed_by')
        book.borrowed_date = data.get('borrowed_date')
        return book
    
    def borrow(self, reader_name: str) -> None:
        """Выдаёт книгу читателю."""
        self.is_borrowed = True
        self.borrowed_by = reader_name
        self.borrowed_date = datetime.now().isoformat()
    
    def return_book(self) -> None:
        """Возвращает книгу в библиотеку."""
        self.is_borrowed = False
        self.borrowed_by = None
        self.borrowed_date = None
    
    def __str__(self) -> str:
        status = "✓ В наличии" if not self.is_borrowed else f"✗ Выдана: {self.borrowed_by}"
        return f"{self.title} — {self.author} ({self.year}) [{self.isbn}] {status}"
Файл src/library/models/reader.py:

python
"""
Модуль модели читателя.
Содержит класс Reader.
"""

from typing import List


class Reader:
    """Модель читателя библиотеки."""
    
    def __init__(self, name: str, email: str, reader_id: str):
        self.name = name
        self.email = email
        self.reader_id = reader_id
        self.borrowed_books: List[str] = []  # список ISBN
    
    def to_dict(self) -> dict:
        """Преобразует читателя в словарь для JSON."""
        return {
            'name': self.name,
            'email': self.email,
            'reader_id': self.reader_id,
            'borrowed_books': self.borrowed_books
        }
    
    @staticmethod
    def from_dict(data: dict) -> 'Reader':
        """Создаёт читателя из словаря."""
        reader = Reader(data['name'], data['email'], data['reader_id'])
        reader.borrowed_books = data.get('borrowed_books', [])
        return reader
    
    def borrow_book(self, isbn: str) -> None:
        """Добавляет книгу к выданным читателю."""
        if isbn not in self.borrowed_books:
            self.borrowed_books.append(isbn)
    
    def return_book(self, isbn: str) -> None:
        """Удаляет книгу из выданных читателю."""
        if isbn in self.borrowed_books:
            self.borrowed_books.remove(isbn)
    
    def __str__(self) -> str:
        return f"{self.name} ({self.email}) ID: {self.reader_id}, книг на руках: {len(self.borrowed_books)}"
Задание 3.3. Создайте модуль репозитория
Файл src/library/repositories/library_repo.py:

python
"""
Модуль репозитория для работы с хранилищем данных.
Использует АБСОЛЮТНЫЙ импорт для доступа к моделям.
"""

import json
from pathlib import Path
from typing import List, Optional

# АБСОЛЮТНЫЙ импорт (от корня проекта)
from src.library.models.book import Book
from src.library.models.reader import Reader


class LibraryRepository:
    """Репозиторий для работы с JSON-файлами."""
    
    def __init__(self, data_dir: Path):
        self.data_dir = data_dir
        self.books_file = data_dir / 'books.json'
        self.readers_file = data_dir / 'readers.json'
        self._ensure_files_exist()
    
    def _ensure_files_exist(self) -> None:
        """Создаёт папку и файлы данных, если их нет."""
        self.data_dir.mkdir(exist_ok=True)
        if not self.books_file.exists():
            self.books_file.write_text('[]', encoding='utf-8')
        if not self.readers_file.exists():
            self.readers_file.write_text('[]', encoding='utf-8')
    
    # ========== РАБОТА С КНИГАМИ ==========
    
    def _load_books(self) -> List[dict]:
        return json.loads(self.books_file.read_text(encoding='utf-8'))
    
    def _save_books(self, books: List[dict]) -> None:
        self.books_file.write_text(
            json.dumps(books, indent=2, ensure_ascii=False), 
            encoding='utf-8'
        )
    
    def get_all_books(self) -> List[Book]:
        """Возвращает список всех книг."""
        books_data = self._load_books()
        return [Book.from_dict(b) for b in books_data]
    
    def save_book(self, book: Book) -> None:
        """Сохраняет новую книгу."""
        books = self._load_books()
        books.append(book.to_dict())
        self._save_books(books)
    
    def update_book(self, isbn: str, updated_book: Book) -> None:
        """Обновляет данные книги по ISBN."""
        books = self._load_books()
        for i, b in enumerate(books):
            if b['isbn'] == isbn:
                books[i] = updated_book.to_dict()
                break
        self._save_books(books)
    
    def find_book_by_isbn(self, isbn: str) -> Optional[Book]:
        """Находит книгу по ISBN."""
        books = self._load_books()
        for b in books:
            if b['isbn'] == isbn:
                return Book.from_dict(b)
        return None
    
    # ========== РАБОТА С ЧИТАТЕЛЯМИ ==========
    
    def _load_readers(self) -> List[dict]:
        return json.loads(self.readers_file.read_text(encoding='utf-8'))
    
    def _save_readers(self, readers: List[dict]) -> None:
        self.readers_file.write_text(
            json.dumps(readers, indent=2, ensure_ascii=False), 
            encoding='utf-8'
        )
    
    def get_all_readers(self) -> List[Reader]:
        """Возвращает список всех читателей."""
        readers_data = self._load_readers()
        return [Reader.from_dict(r) for r in readers_data]
    
    def save_reader(self, reader: Reader) -> None:
        """Сохраняет нового читателя."""
        readers = self._load_readers()
        readers.append(reader.to_dict())
        self._save_readers(readers)
    
    def update_reader(self, reader_id: str, updated_reader: Reader) -> None:
        """Обновляет данные читателя по ID."""
        readers = self._load_readers()
        for i, r in enumerate(readers):
            if r['reader_id'] == reader_id:
                readers[i] = updated_reader.to_dict()
                break
        self._save_readers(readers)
    
    def find_reader_by_id(self, reader_id: str) -> Optional[Reader]:
        """Находит читателя по ID."""
        readers = self._load_readers()
        for r in readers:
            if r['reader_id'] == reader_id:
                return Reader.from_dict(r)
        return None
Задание 3.4. Создайте модуль сервиса
Файл src/library/services/library_service.py:

python
"""
Модуль сервиса библиотеки.
Содержит бизнес-логику приложения.
Использует ОТНОСИТЕЛЬНЫЙ импорт для доступа к репозиторию и моделям.
"""

from typing import List
from pathlib import Path

# ОТНОСИТЕЛЬНЫЕ импорты (внутри пакета)
from ..repositories.library_repo import LibraryRepository
from ..models.book import Book
from ..models.reader import Reader


class LibraryService:
    """Сервис с бизнес-логикой библиотеки."""
    
    def __init__(self, data_dir: Path):
        self.repo = LibraryRepository(data_dir)
    
    def add_book(self, title: str, author: str, year: int, isbn: str) -> Book:
        """Добавляет новую книгу."""
        existing = self.repo.find_book_by_isbn(isbn)
        if existing:
            raise ValueError(f"Книга с ISBN {isbn} уже существует")
        
        book = Book(title, author, year, isbn)
        self.repo.save_book(book)
        return book
    
    def add_reader(self, name: str, email: str, reader_id: str) -> Reader:
        """Добавляет нового читателя."""
        existing = self.repo.find_reader_by_id(reader_id)
        if existing:
            raise ValueError(f"Читатель с ID {reader_id} уже существует")
        
        reader = Reader(name, email, reader_id)
        self.repo.save_reader(reader)
        return reader
    
    def borrow_book(self, isbn: str, reader_id: str) -> Book:
        """Выдаёт книгу читателю."""
        book = self.repo.find_book_by_isbn(isbn)
        if not book:
            raise ValueError(f"Книга с ISBN {isbn} не найдена")
        
        if book.is_borrowed:
            raise ValueError(f"Книга '{book.title}' уже выдана")
        
        reader = self.repo.find_reader_by_id(reader_id)
        if not reader:
            raise ValueError(f"Читатель с ID {reader_id} не найден")
        
        # Обновляем книгу
        book.borrow(reader.name)
        self.repo.update_book(isbn, book)
        
        # Обновляем читателя
        reader.borrow_book(isbn)
        self.repo.update_reader(reader_id, reader)
        
        return book
    
    def return_book(self, isbn: str) -> Book:
        """Возвращает книгу обратно в библиотеку."""
        book = self.repo.find_book_by_isbn(isbn)
        if not book:
            raise ValueError(f"Книга с ISBN {isbn} не найдена")
        
        if not book.is_borrowed:
            raise ValueError(f"Книга '{book.title}' не была выдана")
        
        # Находим читателя, который брал книгу
        readers = self.repo.get_all_readers()
        reader = next((r for r in readers if isbn in r.borrowed_books), None)
        
        # Обновляем книгу
        book.return_book()
        self.repo.update_book(isbn, book)
        
        # Обновляем читателя
        if reader:
            reader.return_book(isbn)
            self.repo.update_reader(reader.reader_id, reader)
        
        return book
    
    def search_books(self, query: str) -> List[Book]:
        """Ищет книги по названию или автору."""
        all_books = self.repo.get_all_books()
        query_lower = query.lower()
        return [
            book for book in all_books
            if query_lower in book.title.lower() or query_lower in book.author.lower()
        ]
    
    def get_available_books(self) -> List[Book]:
        """Возвращает список доступных книг."""
        all_books = self.repo.get_all_books()
        return [book for book in all_books if not book.is_borrowed]
    
    def get_borrowed_books(self) -> List[Book]:
        """Возвращает список выданных книг."""
        all_books = self.repo.get_all_books()
        return [book for book in all_books if book.is_borrowed]
    
    def get_reader_books(self, reader_id: str) -> List[Book]:
        """Возвращает список книг, которые взял читатель."""
        reader = self.repo.find_reader_by_id(reader_id)
        if not reader:
            return []
        
        all_books = self.repo.get_all_books()
        return [book for book in all_books if book.isbn in reader.borrowed_books]
    
    def get_all_books(self) -> List[Book]:
        """Возвращает все книги."""
        return self.repo.get_all_books()
    
    def get_all_readers(self) -> List[Reader]:
        """Возвращает всех читателей."""
        return self.repo.get_all_readers()
Задание 3.5. Создайте модули UI
Файл src/ui/display.py:

python
"""
Модуль отображения данных.
Содержит функции для красивого вывода информации.
"""

from typing import List

# АБСОЛЮТНЫЙ импорт из models
from src.library.models.book import Book
from src.library.models.reader import Reader


def print_header(title: str) -> None:
    """Печатает заголовок с рамкой."""
    print("\n" + "=" * 50)
    print(f"  {title}")
    print("=" * 50)


def print_books(books: List[Book]) -> None:
    """Печатает список книг."""
    if not books:
        print("  Нет книг")
        return
    for book in books:
        print(f"  • {book}")


def print_readers(readers: List[Reader]) -> None:
    """Печатает список читателей."""
    if not readers:
        print("  Нет читателей")
        return
    for reader in readers:
        print(f"  • {reader}")


def print_success(message: str) -> None:
    """Печатает сообщение об успехе."""
    print(f"✓ {message}")


def print_error(message: str) -> None:
    """Печатает сообщение об ошибке."""
    print(f"✗ Ошибка: {message}")
Файл src/ui/menu.py:

python
"""
Модуль меню пользовательского интерфейса.
Содержит основное меню и обработку ввода.
"""

# АБСОЛЮТНЫЙ импорт из services и display
from src.library.services.library_service import LibraryService
from src.ui.display import (
    print_header, print_books, print_readers,
    print_success, print_error
)


def run_menu(service: LibraryService) -> None:
    """Запускает главное меню программы."""
    
    while True:
        print_header("БИБЛИОТЕЧНАЯ СИСТЕМА")
        print("1. Добавить книгу")
        print("2. Добавить читателя")
        print("3. Выдать книгу")
        print("4. Вернуть книгу")
        print("5. Поиск книг")
        print("6. Список всех книг")
        print("7. Доступные книги")
        print("8. Выданные книги")
        print("9. Книги читателя")
        print("10. Список читателей")
        print("11. Выйти")
        
        choice = input("\nВыберите действие: ")
        
        if choice == '1':
            _add_book(service)
        elif choice == '2':
            _add_reader(service)
        elif choice == '3':
            _borrow_book(service)
        elif choice == '4':
            _return_book(service)
        elif choice == '5':
            _search_books(service)
        elif choice == '6':
            _show_all_books(service)
        elif choice == '7':
            _show_available_books(service)
        elif choice == '8':
            _show_borrowed_books(service)
        elif choice == '9':
            _show_reader_books(service)
        elif choice == '10':
            _show_all_readers(service)
        elif choice == '11':
            print("До свидания!")
            break
        else:
            print_error("Неверный выбор, попробуйте снова")


def _add_book(service: LibraryService) -> None:
    """Обработчик добавления книги."""
    print_header("ДОБАВЛЕНИЕ КНИГИ")
    title = input("Название: ")
    author = input("Автор: ")
    year = int(input("Год: "))
    isbn = input("ISBN: ")
    try:
        book = service.add_book(title, author, year, isbn)
        print_success(f"Книга добавлена: {book.title}")
    except ValueError as e:
        print_error(str(e))


def _add_reader(service: LibraryService) -> None:
    """Обработчик добавления читателя."""
    print_header("ДОБАВЛЕНИЕ ЧИТАТЕЛЯ")
    name = input("Имя: ")
    email = input("Email: ")
    reader_id = input("ID читателя: ")
    try:
        reader = service.add_reader(name, email, reader_id)
        print_success(f"Читатель добавлен: {reader.name}")
    except ValueError as e:
        print_error(str(e))


def _borrow_book(service: LibraryService) -> None:
    """Обработчик выдачи книги."""
    print_header("ВЫДАЧА КНИГИ")
    isbn = input("ISBN книги: ")
    reader_id = input("ID читателя: ")
    try:
        book = service.borrow_book(isbn, reader_id)
        print_success(f"Книга '{book.title}' выдана")
    except ValueError as e:
        print_error(str(e))


def _return_book(service: LibraryService) -> None:
    """Обработчик возврата книги."""
    print_header("ВОЗВРАТ КНИГИ")
    isbn = input("ISBN книги: ")
    try:
        book = service.return_book(isbn)
        print_success(f"Книга '{book.title}' возвращена")
    except ValueError as e:
        print_error(str(e))


def _search_books(service: LibraryService) -> None:
    """Обработчик поиска книг."""
    print_header("ПОИСК КНИГ")
    query = input("Поисковый запрос: ")
    books = service.search_books(query)
    print_books(books)


def _show_all_books(service: LibraryService) -> None:
    """Показывает все книги."""
    print_header("ВСЕ КНИГИ")
    books = service.get_all_books()
    print_books(books)


def _show_available_books(service: LibraryService) -> None:
    """Показывает доступные книги."""
    print_header("ДОСТУПНЫЕ КНИГИ")
    books = service.get_available_books()
    print_books(books)


def _show_borrowed_books(service: LibraryService) -> None:
    """Показывает выданные книги."""
    print_header("ВЫДАННЫЕ КНИГИ")
    books = service.get_borrowed_books()
    print_books(books)


def _show_reader_books(service: LibraryService) -> None:
    """Показывает книги читателя."""
    print_header("КНИГИ ЧИТАТЕЛЯ")
    reader_id = input("ID читателя: ")
    books = service.get_reader_books(reader_id)
    print_books(books)


def _show_all_readers(service: LibraryService) -> None:
    """Показывает всех читателей."""
    print_header("ВСЕ ЧИТАТЕЛИ")
    readers = service.get_all_readers()
    print_readers(readers)
Задание 3.6. Создайте главный модуль
Файл main.py (корень проекта):

python
#!/usr/bin/env python3
"""
Главный модуль библиотечной системы.
Точка входа в приложение.
"""

from pathlib import Path

# АБСОЛЮТНЫЙ импорт
from src.library.services.library_service import LibraryService
from src.ui.menu import run_menu


def main():
    """Запускает библиотечную систему."""
    # Определяем папку для данных (кроссплатформенно через pathlib)
    data_dir = Path(__file__).parent / "data"
    
    # Создаём сервис и запускаем меню
    service = LibraryService(data_dir)
    run_menu(service)


if __name__ == "__main__":
    main()
Задание 3.7. Настройте __init__.py для удобного импорта
Файл src/library/__init__.py:

python
"""
Пакет library — основная библиотечная система.
Экспортирует основные классы и сервисы для удобного импорта.
"""

from .models.book import Book
from .models.reader import Reader
from .services.library_service import LibraryService

__all__ = [
    'Book',
    'Reader', 
    'LibraryService'
]
Часть 4. Проверка и тестирование (15 минут)
Задание 4.1. Запустите отрефакторенный проект
bash
# Перейдите в корень проекта
cd library_system

# Запустите программу
python main.py
Ожидаемый результат: программа должна работать так же, как исходный монолит, но с правильно организованной структурой.

Задание 4.2. Проверьте импорты
Создайте временный файл test_imports.py в корне проекта:

python
"""Тестовый скрипт для проверки импортов."""

print("=== Тестирование импортов ===\n")

# Тест 1: импорт через пакет
print("1. Импорт через пакет (src.library):")
from src.library import Book, Reader, LibraryService
print(f"   ✓ Book: {Book}")
print(f"   ✓ Reader: {Reader}")
print(f"   ✓ LibraryService: {LibraryService}")

# Тест 2: импорт конкретных модулей
print("\n2. Импорт конкретных модулей:")
from src.library.models.book import Book as BookDirect
from src.library.models.reader import Reader as ReaderDirect
print(f"   ✓ BookDirect: {BookDirect}")
print(f"   ✓ ReaderDirect: {ReaderDirect}")

# Тест 3: импорт UI
print("\n3. Импорт UI модулей:")
from src.ui.display import print_header, print_success
from src.ui.menu import run_menu
print(f"   ✓ print_header: {print_header}")
print(f"   ✓ run_menu: {run_menu}")

print("\n✅ Все импорты работают корректно!")
Запустите:

bash
python test_imports.py
Контрольные вопросы (для отчёта)
Ответьте письменно:

Какие изменения произошли в коде после рефакторинга?
Перечислите основные отличия от монолита.

Где в новом проекте используются абсолютные импорты, а где — относительные? Приведите примеры.

Почему в library_service.py используются относительные импорты (from ..repositories...), а в menu.py — абсолютные (from src.library.services...)?

Какие преимущества дало выделение моделей Book и Reader в отдельные файлы?

Как изменилась работа с датами после рефакторинга? Где теперь находится логика borrow() и return_book()?

Что будет, если удалить файл src/library/__init__.py? Проверьте экспериментально.

Что сдавать
В отчёт по ПЗ 2.4 включить:

Ответы на вопросы из Задания 1.1 (анализ монолита).

Схему целевой структуры (дерево папок из Задания 2.1).

Таблицу распределения ответственности (Задание 2.2).

Содержимое всех созданных файлов (можно по частям):

src/library/models/book.py

src/library/models/reader.py

src/library/repositories/library_repo.py

src/library/services/library_service.py

src/library/__init__.py

src/ui/display.py

src/ui/menu.py

main.py

Скриншот успешного выполнения python main.py (хотя бы одно действие).

Результат работы python test_imports.py.

Ответы на контрольные вопросы.

Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Рефакторинг не выполнен или программа не работает
3 (удовлетворительно)	Создана структура, но импорты работают не все
4 (хорошо)	Все модули созданы, импорты работают, программа запускается
5 (отлично)	Полностью рабочий проект, корректные импорты, есть __init__.py, код соответствует PEP8
Шпаргалка по рефакторингу
python
# Принципы рефакторинга:

# 1. Один файл → один класс/одна ответственность
# ПЛОХО: всё в одном файле
# ХОРОШО: models/book.py, models/reader.py, services/library_service.py

# 2. Абсолютные импорты для внешних модулей
from src.library.models.book import Book

# 3. Относительные импорты для внутренних связей внутри пакета
from ..models.book import Book

# 4. __init__.py для удобного API пакета
from .models.book import Book
__all__ = ['Book']

# 5. pathlib для кроссплатформенных путей
data_dir = Path(__file__).parent / "data"
Итог практической работы
Вы сегодня:

✅ Проанализировали "плохой" монолитный код

✅ Спланировали структуру для рефакторинга

✅ Разбили монолит на модули и пакеты

✅ Настроили абсолютные и относительные импорты

✅ Создали __init__.py для удобного API

✅ Проверили, что программа работает так же, как и исходная

✅ Получили готовый, поддерживаемый проект

Теперь вы умеете превращать хаотичный код в профессиональную структуру!
