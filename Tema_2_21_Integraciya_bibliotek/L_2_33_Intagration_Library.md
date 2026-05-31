# Тема 2.33. Интеграция программных модулей и внешних библиотек. Поиск, установка, версионирование библиотек из PyPI

**Цель лекции:**  
Изучить методы поиска, установки и управления версиями внешних библиотек из PyPI (Python Package Index), освоить инструменты для работы с зависимостями (pip, pipenv, poetry), научиться интегрировать сторонние модули в Python-проекты.

> Главная мысль: **Не изобретайте велосипед. PyPI содержит сотни тысяч библиотек на все случаи жизни. Главное — уметь их находить и правильно подключать.**

---

## Содержание

1. [Введение в PyPI и экосистему Python](#1-введение-в-pypi-и-экосистему-python)
2. [Поиск библиотек](#2-поиск-библиотек)
3. [Установка библиотек (pip)](#3-установка-библиотек-pip)
4. [Версионирование и управление зависимостями](#4-версионирование-и-управление-зависимостями)
5. [Инструменты для управления зависимостями](#5-инструменты-для-управления-зависимостями)
6. [Практические примеры интеграции](#6-практические-примеры-интеграции)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Введение в PyPI и экосистему Python

### 1.1. Что такое PyPI

**PyPI (Python Package Index)** — официальный репозиторий пакетов для Python.
┌─────────────────────────────────────────────────────────────────────────────┐
│ ЭКОСИСТЕМА PYTHON │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ PyPI (pypi.org) │ │
│ │ Более 400 000 пакетов │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │ │
│ ▼ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ Инструменты │ │
│ │ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │ │
│ │ │ pip │ │ pipenv │ │ poetry │ │ conda │ │ │
│ │ └─────────┘ └─────────┘ └─────────┘ └─────────┘ │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │ │
│ ▼ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ Ваш проект │ │
│ │ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │ │
│ │ │requests │ │numpy │ │fastapi │ │pandas │ │ │
│ │ └─────────┘ └─────────┘ └─────────┘ └─────────┘ │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Популярные категории библиотек

| Категория | Примеры библиотек | Назначение |
|-----------|-------------------|------------|
| **Веб-фреймворки** | Django, Flask, FastAPI | Создание веб-приложений |
| **HTTP-клиенты** | requests, aiohttp, httpx | Работа с API |
| **Наука и анализ** | numpy, pandas, scipy | Работа с данными |
| **Машинное обучение** | tensorflow, pytorch, scikit-learn | ML модели |
| **Базы данных** | SQLAlchemy, psycopg2, asyncpg | ORM, драйверы |
| **Тестирование** | pytest, unittest, mock | Тесты |
| **Визуализация** | matplotlib, seaborn, plotly | Графики |
| **Парсинг** | beautifulsoup4, scrapy, lxml | Парсинг данных |
| **Асинхронность** | aiohttp, asyncio, anyio | Async I/O |
| **GUI** | tkinter, PyQt, Kivy | Графические интерфейсы |

### 1.3. Структура пакета PyPI
┌─────────────────────────────────────────────────────────────────────────────┐
│ СТРУКТУРА ПАКЕТА НА PyPI │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ Название пакета: requests │
│ Версия: 2.31.0 │
│ Автор: Kenneth Reitz │
│ Лицензия: Apache 2.0 │
│ Зависимости: urllib3, certifi, charset-normalizer, idna │
│ │
│ Содержимое: │
│ ├── requests/ # Основной модуль │
│ │ ├── init.py │
│ │ ├── api.py │
│ │ └── ... │
│ ├── tests/ # Тесты │
│ └── setup.py / pyproject.toml # Конфигурация пакета │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

---

## 2. Поиск библиотек

### 2.1. Поиск на сайте PyPI

```bash
# Поиск через веб-интерфейс
https://pypi.org/search/?q=requests

# Информация о пакете
https://pypi.org/project/requests/
2.2. Поиск через командную строку
bash
# Поиск пакетов (требуется pip-search, устаревший)
pip search requests

# Альтернатива: использование pipx
pipx run pypi-search requests

# Поиск через poetry
poetry search requests
2.3. Критерии выбора библиотеки
Критерий	Что проверять	Хороший показатель
Популярность	Количество загрузок/звёзд	> 1 млн/месяц
Активность	Дата последнего обновления	< 6 месяцев
Поддержка	Количество maintainers	> 2
Документация	Наличие README, docs	Есть
Тесты	Наличие тестов	Высокое покрытие
Лицензия	Тип лицензии	MIT, BSD, Apache
Зависимости	Количество зависимостей	Минимум
2.4. Проверка качества пакета
bash
# Проверка уязвимостей
pip install safety
safety check --json

# Проверка лицензий
pip install licensed
licensed cache

# Анализ зависимостей
pip install pipdeptree
pipdeptree
3. Установка библиотек (pip)
3.1. Основные команды pip
bash
# Установка пакета
pip install requests

# Установка конкретной версии
pip install requests==2.31.0

# Установка версии не ниже указанной
pip install 'requests>=2.30.0'

# Установка в диапазоне версий
pip install 'requests>=2.30.0,<3.0.0'

# Установка нескольких пакетов
pip install requests numpy pandas

# Установка из файла зависимостей
pip install -r requirements.txt

# Установка с дополнительными опциями (extras)
pip install 'pydantic[email]'

# Установка в режиме разработки (editable)
pip install -e .
3.2. Управление версиями pip
bash
# Проверка версии pip
pip --version

# Обновление pip
pip install --upgrade pip

# Обновление пакета
pip install --upgrade requests

# Обновление всех пакетов (требуется дополнительный инструмент)
pip install pip-upgrader
pip-upgrade
3.3. Просмотр установленных пакетов
bash
# Список всех установленных пакетов
pip list

# Список с устаревшими версиями
pip list --outdated

# Информация о пакете
pip show requests

# Зависимости пакета
pip show --verbose requests

# Дерево зависимостей
pip install pipdeptree
pipdeptree
3.4. Удаление пакетов
bash
# Удаление пакета
pip uninstall requests

# Удаление нескольких пакетов
pip uninstall requests numpy pandas

# Удаление с подтверждением
pip uninstall -y requests
4. Версионирование и управление зависимостями
4.1. Семантическое версионирование (SemVer)
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                    СЕМАНТИЧЕСКОЕ ВЕРСИОНИРОВАНИЕ                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                         MAJOR.MINOR.PATCH                                   │
│                              │    │    │                                   │
│                              │    │    └── PATCH (исправление ошибок)      │
│                              │    │        • обратно совместимо            │
│                              │    │                                        │
│                              │    └─────── MINOR (новая функциональность)  │
│                              │            • обратно совместимо             │
│                              │                                            │
│                              └──────────── MAJOR (крупные изменения)       │
│                                           • НЕ обратно совместимо          │
│                                                                             │
│   Примеры:                                                                  │
│   • 1.0.0 → 1.0.1 (исправили баг)                                          │
│   • 1.0.0 → 1.1.0 (добавили функцию)                                       │
│   • 1.0.0 → 2.0.0 (сломали совместимость)                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
4.2. Операторы версий в requirements.txt
Оператор	Значение	Пример
==	Точная версия	requests==2.31.0
>=	Выше или равно	requests>=2.30.0
<=	Ниже или равно	requests<=2.32.0
>	Строго выше	requests>2.30.0
<	Строго ниже	requests<3.0.0
~=	Совместимая версия	requests~=2.30.0 (>=2.30.0, <2.31.0)
!=	Не равна	requests!=2.31.0
4.3. Файл requirements.txt
txt
# requirements.txt — зависимости проекта

# Основные зависимости
requests==2.31.0
numpy==1.24.3
pandas==2.0.3

# Зависимости с диапазоном
fastapi>=0.100.0,<0.200.0
uvicorn[standard]>=0.23.0

# Зависимости из Git
git+https://github.com/psf/requests.git@main

# Зависимости с дополнительными опциями
pydantic[email]==2.4.0
4.4. Файл requirements-dev.txt
txt
# requirements-dev.txt — зависимости для разработки

-r requirements.txt  # наследование базовых зависимостей

# Тестирование
pytest==7.4.0
pytest-cov==4.1.0
pytest-asyncio==0.21.0

# Линтеры и форматтеры
black==23.9.0
isort==5.12.0
flake8==6.1.0
mypy==1.5.0

# Отладка
ipdb==0.13.13
ipython==8.15.0
5. Инструменты для управления зависимостями
5.1. Сравнение инструментов
Инструмент	Файлы	Замок версий	Виртуальное окружение	Простота
pip + venv	requirements.txt	❌	✅	Высокая
pip-tools	requirements.in	requirements.txt	❌	Средняя
pipenv	Pipfile	Pipfile.lock	✅	Средняя
poetry	pyproject.toml	poetry.lock	✅	Средняя
conda	environment.yml	✅	✅	Низкая
5.2. pipenv
bash
# Установка
pip install pipenv

# Создание виртуального окружения и установка
pipenv install requests

# Установка в dev-зависимости
pipenv install --dev pytest

# Активация окружения
pipenv shell

# Запуск команды в окружении
pipenv run python script.py

# Установка из Pipfile
pipenv install

# Обновление замка
pipenv lock

# Показать граф зависимостей
pipenv graph
Pipfile:

toml
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
requests = "*"
numpy = "*"

[dev-packages]
pytest = "*"

[requires]
python_version = "3.11"
5.3. Poetry
bash
# Установка
curl -sSL https://install.python-poetry.org | python3 -

# Инициализация проекта
poetry new my-project
cd my-project

# Добавление зависимости
poetry add requests

# Добавление dev-зависимости
poetry add --dev pytest

# Установка всех зависимостей
poetry install

# Активация виртуального окружения
poetry shell

# Запуск команды
poetry run python script.py

# Обновление зависимостей
poetry update

# Показать дерево зависимостей
poetry show --tree
pyproject.toml:

toml
[tool.poetry]
name = "my-project"
version = "0.1.0"
description = "Пример проекта"
authors = ["Ivan Ivanov <ivan@example.com>"]

[tool.poetry.dependencies]
python = "^3.11"
requests = "^2.31.0"
numpy = "^1.24.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4.0"
black = "^23.9.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
5.4. pip-tools
bash
# Установка
pip install pip-tools

# Создание requirements.in
echo "requests" > requirements.in

# Генерация requirements.txt
pip-compile requirements.in

# Обновление
pip-compile --upgrade

# Установка
pip-sync requirements.txt
6. Практические примеры интеграции
6.1. Интеграция HTTP-клиента (requests)
python
import requests
from typing import Dict, Any


class WeatherAPIClient:
    """Клиент для интеграции с погодным API."""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = "https://api.openweathermap.org/data/2.5"
    
    def get_weather(self, city: str) -> Dict[str, Any]:
        """Получение погоды для города."""
        url = f"{self.base_url}/weather"
        params = {
            "q": city,
            "appid": self.api_key,
            "units": "metric",
            "lang": "ru"
        }
        
        response = requests.get(url, params=params, timeout=10)
        response.raise_for_status()
        return response.json()


# Использование
client = WeatherAPIClient(api_key="your-api-key")
weather = client.get_weather("Moscow")
print(f"Температура: {weather['main']['temp']}°C")
6.2. Интеграция с БД (SQLAlchemy)
python
from sqlalchemy import create_engine, Column, Integer, String, Float
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# Установка: pip install sqlalchemy psycopg2-binary

# Подключение к БД
DATABASE_URL = "postgresql://user:password@localhost/dbname"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()


class Product(Base):
    """Модель товара."""
    __tablename__ = "products"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, nullable=False)
    price = Column(Float, nullable=False)
    stock = Column(Integer, default=0)


# Создание таблиц
Base.metadata.create_all(bind=engine)

# Работа с БД
def get_products():
    db = SessionLocal()
    try:
        products = db.query(Product).all()
        return products
    finally:
        db.close()
6.3. Интеграция с обработкой данных (pandas)
python
import pandas as pd
import numpy as np

# Установка: pip install pandas numpy

# Чтение данных
df = pd.read_csv('data.csv')

# Обработка
df['price_usd'] = df['price_rub'] / 90.5
df['total'] = df['price'] * df['quantity']

# Агрегация
stats = df.groupby('category').agg({
    'total': ['sum', 'mean', 'count']
}).round(2)

# Фильтрация
filtered = df[df['price'] > 1000]

# Экспорт
filtered.to_json('output.json', orient='records', force_ascii=False)
6.4. Интеграция с веб-фреймворком (FastAPI)
python
from fastapi import FastAPI, Depends
from pydantic import BaseModel
import uvicorn

# Установка: pip install fastapi uvicorn

app = FastAPI()


class Item(BaseModel):
    name: str
    price: float


@app.get("/")
async def root():
    return {"message": "Hello World"}


@app.post("/items")
async def create_item(item: Item):
    return {"id": 1, "name": item.name, "price": item.price}


if __name__ == "__main__":
    uvicorn.run(app, host="127.0.0.1", port=8000)
6.5. Интеграция с парсингом (BeautifulSoup)
python
import requests
from bs4 import BeautifulSoup

# Установка: pip install beautifulsoup4 lxml

def parse_website(url: str):
    """Парсинг веб-страницы."""
    response = requests.get(url, timeout=10)
    soup = BeautifulSoup(response.text, 'html.parser')
    
    # Извлечение заголовка
    title = soup.find('title').text if soup.find('title') else "No title"
    
    # Извлечение всех ссылок
    links = [a.get('href') for a in soup.find_all('a') if a.get('href')]
    
    return {
        "title": title,
        "links_count": len(links),
        "links": links[:10]  # только первые 10
    }

# Использование
data = parse_website("https://www.python.org")
print(f"Заголовок: {data['title']}")
print(f"Найдено ссылок: {data['links_count']}")
7. Контрольные вопросы
Что такое PyPI и для чего он нужен?

Как установить библиотеку конкретной версии с помощью pip?

В чём разница между requirements.txt и Pipfile?

Что такое семантическое версионирование (SemVer)?

Как обновить все устаревшие пакеты в проекте?

В чём разница между pipenv и poetry?

Как проверить безопасность используемых библиотек?

Что делает команда pip freeze > requirements.txt?

Как установить библиотеку из Git-репозитория?

Как решить конфликт версий зависимостей?

8. Практическое задание
Задание 1 (базовое)
Создайте виртуальное окружение, установите библиотеки requests и pytest, сохраните зависимости в requirements.txt.

Задание 2 (среднее)
Создайте проект с использованием Poetry. Добавьте зависимости: fastapi, uvicorn, requests. Настройте скрипты для запуска.

Задание 3 (сложное)
Интегрируйте в проект три внешние библиотеки:

aiohttp для асинхронных запросов

pydantic для валидации данных

loguru для логирования
Настройте управление зависимостями через pip-tools.

9. Шпаргалка
bash
# === PIP ===
pip install package           # установка
pip install -r requirements.txt  # из файла
pip freeze > requirements.txt    # сохранить
pip list                         # список
pip show package                 # информация
pip uninstall package            # удалить
pip install --upgrade package    # обновить

# === PIPENV ===
pipenv install package        # установка
pipenv install --dev package  # dev-зависимость
pipenv shell                  # активация
pipenv graph                  # дерево зависимостей

# === POETRY ===
poetry add package            # установка
poetry add --dev package      # dev-зависимость
poetry install                # установка всех
poetry shell                  # активация
poetry show --tree            # дерево зависимостей

# === ОПЕРАТОРЫ ВЕРСИЙ ===
package==1.2.3                # точная версия
package>=1.2.0                # минимальная
package<2.0.0                 # максимальная
package~=1.2.0                # совместимая
Итог лекции
Вы сегодня:

✅ Изучили экосистему PyPI и поиск библиотек

✅ Освоили установку и управление пакетами с помощью pip

✅ Изучили семантическое версионирование и управление зависимостями

✅ Освоили современные инструменты: pipenv, poetry, pip-tools

✅ Интегрировали внешние библиотеки в проекты

Теперь вы можете эффективно управлять зависимостями и интегрировать сторонние библиотеки в свои проекты!

