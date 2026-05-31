# ПЗ 2.40. Интеграция внешней библиотеки из PyPI

**Тема:** Интеграция программных модулей, PyPI, управление зависимостями, pip

**Цель работы:**  
Научиться находить, устанавливать и интегрировать внешние библиотеки из PyPI в Python-проекты, управлять зависимостями, обрабатывать ошибки импорта и версионировать библиотеки.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленный pip

> Главная мысль: **PyPI — это огромная экосистема готовых решений. Умение находить и интегрировать нужные библиотеки ускоряет разработку в десятки раз.**

---

## Содержание

1. [Теоретическая справка](#1-теоретическая-справка)
2. [Нулевой вариант (эталонный)](#2-нулевой-вариант-эталонный)
3. [25 вариантов практической работы](#3-25-вариантов-практической-работы)
4. [Критерии оценки](#4-критерии-оценки)
5. [Шпаргалка](#5-шпаргалка)

---

## 1. Теоретическая справка

### 1.1. Что такое PyPI

**PyPI (Python Package Index)** — официальный репозиторий пакетов для Python.
┌─────────────────────────────────────────────────────────────────────────────┐
│ ПРОЦЕСС ИНТЕГРАЦИИ БИБЛИОТЕКИ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ШАГ 1: ПОИСК │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ • pypi.org/search │ │
│ │ • pip search (устаревший) │ │
│ │ • Google "python библиотека для ..." │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │ │
│ ▼ │
│ ШАГ 2: УСТАНОВКА │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ • pip install package │ │
│ │ • pip install package==version │ │
│ │ • pip install -r requirements.txt │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │ │
│ ▼ │
│ ШАГ 3: ИНТЕГРАЦИЯ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ • import package │ │
│ │ • Использование API библиотеки │ │
│ │ • Обработка ошибок импорта │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │ │
│ ▼ │
│ ШАГ 4: УПРАВЛЕНИЕ ЗАВИСИМОСТЯМИ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ • Сохранение в requirements.txt │ │
│ │ • Фиксация версий │ │
│ │ • Обновление библиотек │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Критерии выбора библиотеки

| Критерий | Что проверять | Хороший показатель |
|----------|---------------|---------------------|
| **Популярность** | Downloads/month | > 100 000 |
| **Активность** | Последний коммит | < 6 месяцев |
| **Документация** | README, docs | Есть, понятная |
| **Лицензия** | MIT, BSD, Apache | Свободная |
| **Зависимости** | Количество | Минимум |
| **Поддержка Python** | Версии Python | 3.8+ |

### 1.3. Обработка ошибок импорта

```python
try:
    import requests
except ImportError:
    print("Библиотека requests не установлена. Выполните: pip install requests")
    raise

# С проверкой версии
import importlib.metadata

try:
    version = importlib.metadata.version('requests')
    print(f"Установлена версия requests: {version}")
except importlib.metadata.PackageNotFoundError:
    print("Библиотека requests не установлена")
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая интеграцию внешней библиотеки из PyPI.

Техническое задание (нулевой вариант)
Разработайте модуль email_sender.py, который интегрирует внешнюю библиотеку yagmail для отправки email. Требования:

Установка библиотеки через pip

Обработка ошибок импорта

Конфигурация через переменные окружения

Функция отправки email с вложениями

Логирование результатов

Эталонная реализация
python
#!/usr/bin/env python3
"""
email_sender.py — Интеграция библиотеки yagmail для отправки email.
"""

import os
import sys
import logging
from pathlib import Path
from typing import List, Optional

# ============================================================
# НАСТРОЙКА ЛОГИРОВАНИЯ
# ============================================================

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)


# ============================================================
# ПРОВЕРКА НАЛИЧИЯ БИБЛИОТЕКИ
# ============================================================

try:
    import yagmail
    import keyring  # для безопасного хранения паролей
    YAGMAIL_AVAILABLE = True
    logger.info("Библиотека yagmail успешно загружена")
except ImportError as e:
    YAGMAIL_AVAILABLE = False
    logger.error(f"Библиотека не найдена: {e}")
    logger.error("Установите: pip install yagmail keyring")
    print("\n❌ ОШИБКА: Требуемые библиотеки не установлены")
    print("   Выполните: pip install yagmail keyring\n")
    sys.exit(1)


# ============================================================
# ПРОВЕРКА ВЕРСИИ
# ============================================================

try:
    import importlib.metadata
    version = importlib.metadata.version('yagmail')
    logger.info(f"Версия yagmail: {version}")
except Exception:
    logger.warning("Не удалось определить версию yagmail")


# ============================================================
# КЛАСС ДЛЯ ОТПРАВКИ EMAIL
# ============================================================

class EmailSender:
    """
    Класс для отправки email с использованием библиотеки yagmail.
    
    Использование:
        sender = EmailSender()
        sender.send(
            to="recipient@example.com",
            subject="Привет",
            contents="Текст письма",
            attachments=["file.pdf"]
        )
    """
    
    def __init__(
        self,
        user: Optional[str] = None,
        password: Optional[str] = None,
        smtp_config: Optional[dict] = None
    ):
        """
        Инициализация отправителя email.
        
        Args:
            user: Email отправителя (по умолчанию из env YAGMAIL_USER)
            password: Пароль отправителя (по умолчанию из env YAGMAIL_PASSWORD)
            smtp_config: Конфигурация SMTP (хост, порт)
        """
        self.user = user or os.getenv('YAGMAIL_USER')
        self.password = password or os.getenv('YAGMAIL_PASSWORD')
        
        if not self.user or not self.password:
            raise ValueError(
                "Не указаны учетные данные. Установите переменные окружения:\n"
                "  export YAGMAIL_USER='your@email.com'\n"
                "  export YAGMAIL_PASSWORD='your_password'"
            )
        
        self.smtp_config = smtp_config or {
            'host': os.getenv('SMTP_HOST', 'smtp.gmail.com'),
            'port': int(os.getenv('SMTP_PORT', '587'))
        }
        
        self._yag = None
    
    def _get_client(self) -> 'yagmail.SMTP':
        """Создание или получение клиента yagmail."""
        if self._yag is None:
            self._yag = yagmail.SMTP(
                user=self.user,
                password=self.password,
                host=self.smtp_config['host'],
                port=self.smtp_config['port'],
                smtp_starttls=True
            )
        return self._yag
    
    def send(
        self,
        to: str,
        subject: str,
        contents: str,
        attachments: Optional[List[str]] = None,
        cc: Optional[List[str]] = None,
        bcc: Optional[List[str]] = None
    ) -> bool:
        """
        Отправка email.
        
        Args:
            to: Получатель (email или список)
            subject: Тема письма
            contents: Текст письма
            attachments: Список путей к файлам для вложения
            cc: Копии
            bcc: Скрытые копии
        
        Returns:
            True если успешно, иначе False
        """
        try:
            yag = self._get_client()
            
            # Обработка вложений
            if attachments:
                # Проверка существования файлов
                valid_attachments = []
                for file_path in attachments:
                    path = Path(file_path)
                    if path.exists():
                        valid_attachments.append(str(path))
                    else:
                        logger.warning(f"Файл не найден: {file_path}")
                
                if valid_attachments:
                    attachments = valid_attachments
            
            # Отправка
            yag.send(
                to=to,
                subject=subject,
                contents=contents,
                attachments=attachments,
                cc=cc,
                bcc=bcc
            )
            
            logger.info(f"Email успешно отправлен: {subject} -> {to}")
            return True
            
        except Exception as e:
            logger.error(f"Ошибка отправки email: {e}")
            return False
    
    def close(self):
        """Закрытие соединения с SMTP-сервером."""
        if self._yag:
            self._yag.close()
            self._yag = None
    
    def __enter__(self):
        return self
    
    def __exit__(self, *args):
        self.close()


# ============================================================
# ПРИМЕР ИСПОЛЬЗОВАНИЯ
# ============================================================

def create_sample_files():
    """Создание тестовых файлов для вложений."""
    sample_dir = Path("sample_attachments")
    sample_dir.mkdir(exist_ok=True)
    
    # Текстовый файл
    txt_file = sample_dir / "message.txt"
    txt_file.write_text("Это тестовое вложение.", encoding='utf-8')
    
    # JSON файл
    json_file = sample_dir / "data.json"
    import json
    json_file.write_text(
        json.dumps({"name": "Test", "value": 42}, indent=2),
        encoding='utf-8'
    )
    
    return [str(txt_file), str(json_file)]


def main():
    """Демонстрация работы EmailSender."""
    print("=" * 60)
    print("ИНТЕГРАЦИЯ БИБЛИОТЕКИ YAGMAIL")
    print("=" * 60)
    
    # Проверка наличия учётных данных
    user = os.getenv('YAGMAIL_USER')
    password = os.getenv('YAGMAIL_PASSWORD')
    
    if not user or not password:
        print("\n⚠️ Для демонстрации требуется настройка учётных данных.")
        print("   Установите переменные окружения:")
        print("   export YAGMAIL_USER='your@email.com'")
        print("   export YAGMAIL_PASSWORD='your_password'\n")
        print("   Или используйте тестовый режим (без реальной отправки)\n")
        
        # Демонстрация в тестовом режиме
        print("🧪 ТЕСТОВЫЙ РЕЖИМ (без реальной отправки)")
        print("-" * 40)
        
        # Создание тестовых файлов
        attachments = create_sample_files()
        print(f"📎 Созданы тестовые вложения: {attachments}")
        
        print("\n📧 Пример отправки:")
        print(f"   Кому: test@example.com")
        print(f"   Тема: Тестовое письмо из Python")
        print(f"   Текст: Привет! Это тестовое письмо.")
        print(f"   Вложения: {[Path(f).name for f in attachments]}")
        
        print("\n✅ Демонстрация завершена (реальная отправка не производилась)")
        return
    
    # Реальная отправка
    try:
        # Создание вложений
        attachments = create_sample_files()
        
        # Отправка email
        with EmailSender() as sender:
            success = sender.send(
                to=user,  # отправка самому себе
                subject="Тестовое письмо из Python (yagmail)",
                contents="""
                <html>
                <body>
                    <h1>Привет!</h1>
                    <p>Это тестовое письмо, отправленное через <b>yagmail</b>.</p>
                    <p>Библиотека успешно интегрирована!</p>
                    <ul>
                        <li>✅ Установка через pip</li>
                        <li>✅ Интеграция в Python</li>
                        <li>✅ Отправка с вложениями</li>
                    </ul>
                    <p>Время отправки: <i>{}</i></p>
                </body>
                </html>
                """,
                attachments=attachments
            )
            
            if success:
                print("\n✅ EMAIL УСПЕШНО ОТПРАВЛЕН!")
                print(f"   Проверьте почтовый ящик: {user}")
            else:
                print("\n❌ Ошибка при отправке email")
                
    except Exception as e:
        print(f"\n❌ Ошибка: {e}")


if __name__ == "__main__":
    main()
Инструкция по установке и запуску
bash
# 1. Создание виртуального окружения
python -m venv venv
source venv/bin/activate  # Linux/macOS
venv\Scripts\activate     # Windows

# 2. Установка библиотек
pip install yagmail keyring

# 3. Установка переменных окружения (для реальной отправки)
export YAGMAIL_USER='your@gmail.com'
export YAGMAIL_PASSWORD='your_app_password'

# 4. Запуск
python email_sender.py
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (одна библиотека, простая интеграция)

Варианты 9-17: средний (несколько библиотек, конфигурация)

Варианты 18-25: сложный (асинхронные библиотеки, обработка ошибок)

Варианты 1-8 (Базовый уровень)
№	Библиотека	Назначение	Интеграция
1	requests	HTTP-запросы	GET, POST
2	python-dotenv	Переменные окружения	Загрузка .env
3	pytz	Часовые пояса	Конвертация времени
4	python-dateutil	Работа с датами	Парсинг, вычисления
5	emoji	Эмодзи	Добавление эмодзи в текст
6	qrcode	QR-коды	Генерация QR-кодов
7	Pillow	Работа с изображениями	Изменение размера
8	tqdm	Прогресс-бары	Отображение прогресса
Варианты 9-17 (Средний уровень)
№	Библиотеки	Назначение	Особенности
9	pandas + openpyxl	Работа с Excel	Чтение/запись
10	requests + beautifulsoup4	Парсинг	Извлечение данных
11	click + rich	CLI-интерфейс	Красивый вывод
12	fastapi + uvicorn	Веб-API	Создание сервера
13	loguru + colorlog	Логирование	Цветные логи
14	pydantic + email-validator	Валидация	Проверка email
15	redis + pickle	Кэширование	Хранение данных
16	aiohttp + asyncio	Асинхронные запросы	Параллельные запросы
17	sqlalchemy + alembic	Базы данных	Миграции
Варианты 18-25 (Сложный уровень)
№	Библиотеки	Назначение	Особенности
18	fastapi + pydantic + sqlalchemy	Полный веб-стек	REST API + БД
19	scrapy + selenium	Парсинг JS-страниц	Динамический контент
20	celery + redis	Фоновые задачи	Асинхронная обработка
21	pydantic + pytest	Тестирование	Валидация тестов
22	tensorflow + numpy	Машинное обучение	Прогнозирование
23	playwright + pytest	Автоматизация браузера	E2E-тесты
24	grpcio + protobuf	gRPC	Микросервисы
25	prometheus_client + grafana	Мониторинг	Метрики
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Библиотека не интегрирована
3 (удовлетворительно)	Установка и базовое использование
4 (хорошо)	+ обработка ошибок, конфигурация
5 (отлично)	+ логирование, тесты, документация
5. Шпаргалка
bash
# === УСТАНОВКА ===
pip install package_name
pip install package_name==1.2.3
pip install package_name>=1.0.0

# === УПРАВЛЕНИЕ ЗАВИСИМОСТЯМИ ===
pip freeze > requirements.txt
pip install -r requirements.txt

# === ПРОСМОТР ===
pip list
pip show package_name

# === ОБНОВЛЕНИЕ ===
pip install --upgrade package_name
python
# === ИМПОРТ С ОБРАБОТКОЙ ОШИБОК ===
try:
    import library
    LIBRARY_AVAILABLE = True
except ImportError:
    LIBRARY_AVAILABLE = False
    print("Установите: pip install library")

# === ПРОВЕРКА ВЕРСИИ ===
import importlib.metadata
try:
    version = importlib.metadata.version('library')
except importlib.metadata.PackageNotFoundError:
    version = "not installed"

# === КОНФИГУРАЦИЯ ===
import os
from dotenv import load_dotenv

load_dotenv()
API_KEY = os.getenv('API_KEY')
Карточка студента
text
ПЗ 2.40. ИНТЕГРАЦИЯ ВНЕШНЕЙ БИБЛИОТЕКИ ИЗ PYPI

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

=== ИНФОРМАЦИЯ О БИБЛИОТЕКЕ ===

Название: _____________
Версия: _____________
Лицензия: _____________
Ссылка на PyPI: _____________

=== ЭТАПЫ ИНТЕГРАЦИИ ===

□ Поиск и выбор библиотеки
□ Установка через pip
□ Обработка ошибок импорта
□ Базовая настройка
□ Использование API
□ Сохранение зависимостей

=== ОТЧЁТ ===

Файл: _____________
Команда установки: _____________
requirements.txt: _____________

Дата выполнения: _____________
