# Тема 2.39. Конфигурирование приложения: переменные окружения, файлы .env, библиотека python-dotenv

**Цель лекции:**  
Изучить методы конфигурирования Python-приложений, освоить работу с переменными окружения, файлами `.env` и библиотекой `python-dotenv`, научиться создавать безопасные и гибкие конфигурации.

> Главная мысль: **Никаких паролей и ключей в коде! Конфигурация должна быть вне кода — в переменных окружения и файлах .env.**

---

## Содержание

1. [Введение в конфигурирование](#1-введение-в-конфигурирование)
2. [Переменные окружения](#2-переменные-окружения)
3. [Библиотека python-dotenv](#3-библиотека-python-dotenv)
4. [Продвинутая работа с конфигурацией](#4-продвинутая-работа-с-конфигурацией)
5. [Безопасность и лучшие практики](#5-безопасность-и-лучшие-практики)
6. [Практические примеры](#6-практические-примеры)
7. [Контрольные вопросы](#7-контрольные-вопросы)
8. [Практическое задание](#8-практическое-задание)
9. [Шпаргалка](#9-шпаргалка)

---

## 1. Введение в конфигурирование

### 1.1. Почему конфигурация должна быть вне кода
┌─────────────────────────────────────────────────────────────────────────────┐
│ ПРОБЛЕМЫ КОНФИГУРАЦИИ В КОДЕ │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ❌ ПЛОХО: │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ DB_PASSWORD = "my_secret_password" │ │
│ │ API_KEY = "abc123" │ │
│ │ DEBUG = True │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
│ Проблемы: │
│ • Секреты попадают в Git │
│ • Разные окружения требуют изменения кода │
│ • Невозможно переиспользовать код │
│ │
│ ✅ ХОРОШО: │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ DB_PASSWORD = os.getenv('DB_PASSWORD') │ │
│ │ API_KEY = os.getenv('API_KEY') │ │
│ │ DEBUG = os.getenv('DEBUG', 'False').lower() == 'true' │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 1.2. Источники конфигурации

| Источник | Описание | Приоритет |
|----------|----------|-----------|
| **Переменные окружения** | Системные переменные | Высокий |
| **Файлы .env** | Локальные файлы конфигурации | Средний |
| **Файлы конфигурации** | JSON, YAML, TOML | Низкий |
| **Значения по умолчанию** | В коде | Самый низкий |

### 1.3. Принцип 12-factor app
┌─────────────────────────────────────────────────────────────────────────────┐
│ ПРИНЦИП 12-FACTOR APP │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ III. CONFIG — Хранение конфигурации в переменных окружения │
│ │
│ • Конфигурация отделена от кода │
│ • Одинаковый код для разных окружений │
│ • Секреты не попадают в репозиторий │
│ │
│ dev.env staging.env prod.env │
│ │ │ │ │
│ └────────────────┼──────────────────┘ │
│ ▼ │
│ ┌─────────────────┐ │
│ │ ОДИН КОД │ │
│ └─────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

---

## 2. Переменные окружения

### 2.1. Работа с os.environ

```python
import os

# Получение переменной (с проверкой)
db_host = os.environ.get('DB_HOST', 'localhost')
db_port = int(os.environ.get('DB_PORT', '5432'))
debug = os.environ.get('DEBUG', 'False').lower() == 'true'

# Установка переменной (в текущем процессе)
os.environ['MY_VAR'] = 'my_value'

# Проверка существования
if 'API_KEY' in os.environ:
    api_key = os.environ['API_KEY']
else:
    raise ValueError('API_KEY not set')

# Получение всех переменных
for key, value in os.environ.items():
    print(f"{key}={value}")
2.2. Типизация значений
python
import os
from typing import Optional, List, Dict


class EnvParser:
    """Парсер переменных окружения с типизацией."""
    
    @staticmethod
    def get_str(key: str, default: Optional[str] = None) -> str:
        value = os.environ.get(key)
        if value is None:
            if default is not None:
                return default
            raise ValueError(f"Environment variable {key} not set")
        return value
    
    @staticmethod
    def get_int(key: str, default: Optional[int] = None) -> int:
        value = os.environ.get(key)
        if value is None:
            if default is not None:
                return default
            raise ValueError(f"Environment variable {key} not set")
        try:
            return int(value)
        except ValueError:
            raise ValueError(f"Environment variable {key} must be an integer")
    
    @staticmethod
    def get_float(key: str, default: Optional[float] = None) -> float:
        value = os.environ.get(key)
        if value is None:
            if default is not None:
                return default
            raise ValueError(f"Environment variable {key} not set")
        try:
            return float(value)
        except ValueError:
            raise ValueError(f"Environment variable {key} must be a float")
    
    @staticmethod
    def get_bool(key: str, default: Optional[bool] = None) -> bool:
        value = os.environ.get(key)
        if value is None:
            if default is not None:
                return default
            raise ValueError(f"Environment variable {key} not set")
        return value.lower() in ('true', '1', 'yes', 'on')
    
    @staticmethod
    def get_list(key: str, separator: str = ',', default: Optional[List] = None) -> List:
        value = os.environ.get(key)
        if value is None:
            if default is not None:
                return default
            return []
        return [item.strip() for item in value.split(separator) if item.strip()]
    
    @staticmethod
    def get_dict(key: str, default: Optional[Dict] = None) -> Dict:
        import json
        value = os.environ.get(key)
        if value is None:
            return default or {}
        try:
            return json.loads(value)
        except json.JSONDecodeError:
            raise ValueError(f"Environment variable {key} must be a valid JSON")


# Использование
DB_HOST = EnvParser.get_str('DB_HOST', 'localhost')
DB_PORT = EnvParser.get_int('DB_PORT', 5432)
DEBUG = EnvParser.get_bool('DEBUG', False)
ALLOWED_HOSTS = EnvParser.get_list('ALLOWED_HOSTS', ',', ['localhost'])
2.3. Установка переменных окружения
bash
# Linux/macOS
export DB_HOST=localhost
export DB_PORT=5432
export DEBUG=true
python app.py

# Windows (CMD)
set DB_HOST=localhost
set DB_PORT=5432
set DEBUG=true
python app.py

# Windows (PowerShell)
$env:DB_HOST="localhost"
$env:DB_PORT="5432"
$env:DEBUG="true"
python app.py

# Одной строкой
DB_HOST=localhost DB_PORT=5432 python app.py
3. Библиотека python-dotenv
3.1. Установка и базовое использование
bash
pip install python-dotenv
python
from dotenv import load_dotenv
import os

# Загрузка переменных из .env файла
load_dotenv()  # ищет .env в текущей директории

# Теперь переменные доступны через os.environ
db_host = os.getenv('DB_HOST')
db_port = os.getenv('DB_PORT')
secret_key = os.getenv('SECRET_KEY')
3.2. Файл .env
bash
# .env — файл с переменными окружения (НЕ КОММИТИТЬ В GIT!)

# Комментарии начинаются с #
APP_NAME=MyApp
APP_VERSION=1.0.0
DEBUG=true

# База данных
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=postgres
DB_PASSWORD=secret_password

# API ключи
API_KEY=abc123xyz
SECRET_KEY=your-secret-key-here

# Списки
ALLOWED_HOSTS=localhost,127.0.0.1,example.com

# JSON
REDIS_CONFIG={"host": "localhost", "port": 6379}
3.3. Разные файлы для разных окружений
python
from dotenv import load_dotenv
import os

# Определение окружения
env = os.getenv('APP_ENV', 'development')

# Загрузка соответствующего .env файла
if env == 'production':
    load_dotenv('.env.prod')
elif env == 'staging':
    load_dotenv('.env.staging')
else:
    load_dotenv('.env.dev')

# Переопределение переменными окружения (более высокий приоритет)
load_dotenv(override=True)
bash
# .env.dev
DEBUG=true
DB_HOST=localhost
DB_NAME=myapp_dev

# .env.prod
DEBUG=false
DB_HOST=postgres.prod.example.com
DB_NAME=myapp_prod
DB_PASSWORD=secure_password
3.4. Поиск .env файла
python
from dotenv import load_dotenv, find_dotenv
from pathlib import Path

# Поиск .env файла в родительских директориях
dotenv_path = find_dotenv()
print(f"Found .env at: {dotenv_path}")
load_dotenv(dotenv_path)

# Указание конкретного пути
dotenv_path = Path(__file__).parent / '.env'
load_dotenv(dotenv_path)

# Загрузка из строки
from dotenv import dotenv_values

config = dotenv_values(".env")
print(config['DB_HOST'])
4. Продвинутая работа с конфигурацией
4.1. Класс конфигурации с валидацией
python
from dotenv import load_dotenv
import os
from typing import Optional, List
from dataclasses import dataclass, field


@dataclass
class DatabaseConfig:
    """Конфигурация базы данных."""
    host: str = "localhost"
    port: int = 5432
    name: str = "app"
    user: str = "postgres"
    password: str = ""
    pool_size: int = 10
    
    @property
    def url(self) -> str:
        return f"postgresql://{self.user}:{self.password}@{self.host}:{self.port}/{self.name}"


@dataclass
class RedisConfig:
    """Конфигурация Redis."""
    host: str = "localhost"
    port: int = 6379
    db: int = 0
    password: str = ""
    
    @property
    def url(self) -> str:
        if self.password:
            return f"redis://:{self.password}@{self.host}:{self.port}/{self.db}"
        return f"redis://{self.host}:{self.port}/{self.db}"


@dataclass
class AppConfig:
    """Основная конфигурация приложения."""
    
    # Приложение
    app_name: str = "MyApp"
    app_env: str = "development"
    debug: bool = True
    secret_key: str = ""
    
    # Базы данных
    database: DatabaseConfig = field(default_factory=DatabaseConfig)
    redis: RedisConfig = field(default_factory=RedisConfig)
    
    # API
    api_keys: List[str] = field(default_factory=list)
    allowed_hosts: List[str] = field(default_factory=lambda: ["localhost"])
    
    # Логирование
    log_level: str = "INFO"
    log_file: Optional[str] = None
    
    @classmethod
    def from_env(cls, env_file: Optional[str] = None) -> 'AppConfig':
        """Создание конфигурации из переменных окружения."""
        if env_file:
            load_dotenv(env_file)
        else:
            load_dotenv()
        
        # Основные настройки
        config = cls(
            app_name=os.getenv('APP_NAME', 'MyApp'),
            app_env=os.getenv('APP_ENV', 'development'),
            debug=os.getenv('DEBUG', 'false').lower() == 'true',
            secret_key=os.getenv('SECRET_KEY', ''),
            log_level=os.getenv('LOG_LEVEL', 'INFO'),
            log_file=os.getenv('LOG_FILE'),
            allowed_hosts=os.getenv('ALLOWED_HOSTS', 'localhost').split(','),
            api_keys=os.getenv('API_KEYS', '').split(',') if os.getenv('API_KEYS') else []
        )
        
        # Настройки базы данных
        config.database = DatabaseConfig(
            host=os.getenv('DB_HOST', 'localhost'),
            port=int(os.getenv('DB_PORT', '5432')),
            name=os.getenv('DB_NAME', 'app'),
            user=os.getenv('DB_USER', 'postgres'),
            password=os.getenv('DB_PASSWORD', ''),
            pool_size=int(os.getenv('DB_POOL_SIZE', '10'))
        )
        
        # Настройки Redis
        config.redis = RedisConfig(
            host=os.getenv('REDIS_HOST', 'localhost'),
            port=int(os.getenv('REDIS_PORT', '6379')),
            db=int(os.getenv('REDIS_DB', '0')),
            password=os.getenv('REDIS_PASSWORD', '')
        )
        
        return config
    
    def validate(self) -> bool:
        """Валидация конфигурации."""
        errors = []
        
        if self.app_env not in ['development', 'staging', 'production']:
            errors.append(f"Invalid APP_ENV: {self.app_env}")
        
        if self.app_env == 'production' and not self.secret_key:
            errors.append("SECRET_KEY is required in production")
        
        if self.database.password == '' and self.app_env == 'production':
            errors.append("DB_PASSWORD is required in production")
        
        if errors:
            raise ValueError(f"Configuration errors: {', '.join(errors)}")
        
        return True


# Использование
config = AppConfig.from_env()
config.validate()

print(f"App: {config.app_name} ({config.app_env})")
print(f"Database URL: {config.database.url}")
print(f"Redis URL: {config.redis.url}")
4.2. Pydantic для конфигурации
python
from pydantic import BaseSettings, Field, validator
from typing import List, Optional
import os


class Settings(BaseSettings):
    """Настройки приложения с валидацией Pydantic."""
    
    # Приложение
    app_name: str = Field("MyApp", env="APP_NAME")
    app_env: str = Field("development", env="APP_ENV")
    debug: bool = Field(False, env="DEBUG")
    secret_key: str = Field(..., env="SECRET_KEY")
    
    # База данных
    db_host: str = Field("localhost", env="DB_HOST")
    db_port: int = Field(5432, env="DB_PORT")
    db_name: str = Field("app", env="DB_NAME")
    db_user: str = Field("postgres", env="DB_USER")
    db_password: str = Field("", env="DB_PASSWORD")
    
    # Redis
    redis_host: str = Field("localhost", env="REDIS_HOST")
    redis_port: int = Field(6379, env="REDIS_PORT")
    redis_db: int = Field(0, env="REDIS_DB")
    redis_password: Optional[str] = Field(None, env="REDIS_PASSWORD")
    
    # API
    allowed_hosts: List[str] = Field(["localhost"], env="ALLOWED_HOSTS")
    cors_origins: List[str] = Field(["*"], env="CORS_ORIGINS")
    
    @property
    def database_url(self) -> str:
        return f"postgresql://{self.db_user}:{self.db_password}@{self.db_host}:{self.db_port}/{self.db_name}"
    
    @property
    def redis_url(self) -> str:
        if self.redis_password:
            return f"redis://:{self.redis_password}@{self.redis_host}:{self.redis_port}/{self.redis_db}"
        return f"redis://{self.redis_host}:{self.redis_port}/{self.redis_db}"
    
    @validator('app_env')
    def validate_env(cls, v):
        if v not in ['development', 'staging', 'production']:
            raise ValueError(f'Invalid environment: {v}')
        return v
    
    @validator('secret_key')
    def validate_secret_key(cls, v, values):
        if values.get('app_env') == 'production' and not v:
            raise ValueError('SECRET_KEY is required in production')
        return v
    
    class Config:
        env_file = '.env'
        env_file_encoding = 'utf-8'
        case_sensitive = False


# Использование
settings = Settings()
print(f"App: {settings.app_name} ({settings.app_env})")
print(f"Database URL: {settings.database_url}")
5. Безопасность и лучшие практики
5.1. .gitignore для .env файлов
gitignore
# .gitignore

# Файлы с секретами
.env
.env.local
.env.*.local

# Файлы конфигурации с паролями
*.env
*.key
*.pem
secrets/
5.2. Шаблон .env.example
bash
# .env.example — шаблон для команды (коммитится в Git)

# Приложение
APP_NAME=MyApp
APP_ENV=development
DEBUG=true
SECRET_KEY=change_me_in_production

# База данных
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=postgres
DB_PASSWORD=change_me

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0
REDIS_PASSWORD=

# API
ALLOWED_HOSTS=localhost,127.0.0.1
CORS_ORIGINS=http://localhost:3000,http://localhost:8000
5.3. Проверка конфигурации при запуске
python
import os
import sys
from typing import List


class ConfigChecker:
    """Проверка конфигурации при запуске."""
    
    REQUIRED_VARS = [
        'SECRET_KEY',
        'DB_HOST',
        'DB_USER',
        'DB_PASSWORD',
    ]
    
    PRODUCTION_REQUIRED = [
        'SECRET_KEY',
        'DB_PASSWORD',
        'REDIS_PASSWORD',
    ]
    
    @classmethod
    def check(cls) -> List[str]:
        """Проверка наличия обязательных переменных."""
        missing = []
        env = os.getenv('APP_ENV', 'development')
        
        # Основные переменные
        for var in cls.REQUIRED_VARS:
            if not os.getenv(var):
                missing.append(var)
        
        # Дополнительные проверки для production
        if env == 'production':
            for var in cls.PRODUCTION_REQUIRED:
                if not os.getenv(var):
                    missing.append(f"{var} (required in production)")
            
            # Проверка SECRET_KEY на безопасность
            secret_key = os.getenv('SECRET_KEY', '')
            if len(secret_key) < 32:
                missing.append("SECRET_KEY must be at least 32 characters in production")
        
        return missing
    
    @classmethod
    def assert_config(cls):
        """Проверка с остановкой при ошибках."""
        missing = cls.check()
        if missing:
            print("❌ Configuration errors:")
            for var in missing:
                print(f"   - {var}")
            sys.exit(1)
        print("✅ Configuration OK")


# Использование
ConfigChecker.assert_config()
5.4. Генерация секретных ключей
python
import secrets
import string


def generate_secret_key(length: int = 50) -> str:
    """Генерация безопасного секретного ключа."""
    alphabet = string.ascii_letters + string.digits + "!@#$%^&*()"
    return ''.join(secrets.choice(alphabet) for _ in range(length))


def generate_api_key(prefix: str = "", length: int = 32) -> str:
    """Генерация API ключа."""
    key = secrets.token_urlsafe(length)
    return f"{prefix}{key}" if prefix else key


def generate_db_password(length: int = 20) -> str:
    """Генерация пароля для базы данных."""
    alphabet = string.ascii_letters + string.digits
    return ''.join(secrets.choice(alphabet) for _ in range(length))


# Использование
print(f"SECRET_KEY={generate_secret_key()}")
print(f"API_KEY={generate_api_key('sk_')}")
print(f"DB_PASSWORD={generate_db_password()}")
6. Практические примеры
6.1. Конфигурация для веб-приложения
python
# config.py
from dotenv import load_dotenv
import os
from pathlib import Path
from typing import Optional

# Загрузка .env
load_dotenv()

# Базовые пути
BASE_DIR = Path(__file__).parent
DATA_DIR = BASE_DIR / "data"
LOG_DIR = BASE_DIR / "logs"

# Создание необходимых директорий
DATA_DIR.mkdir(exist_ok=True)
LOG_DIR.mkdir(exist_ok=True)


class Config:
    """Базовый класс конфигурации."""
    
    # Приложение
    APP_NAME = os.getenv('APP_NAME', 'MyApp')
    APP_ENV = os.getenv('APP_ENV', 'development')
    DEBUG = os.getenv('DEBUG', 'false').lower() == 'true'
    SECRET_KEY = os.getenv('SECRET_KEY', 'dev-secret-key-change-me')
    
    # Сервер
    HOST = os.getenv('HOST', '127.0.0.1')
    PORT = int(os.getenv('PORT', '8000'))
    WORKERS = int(os.getenv('WORKERS', '1'))
    
    # База данных
    DB_HOST = os.getenv('DB_HOST', 'localhost')
    DB_PORT = int(os.getenv('DB_PORT', '5432'))
    DB_NAME = os.getenv('DB_NAME', 'myapp')
    DB_USER = os.getenv('DB_USER', 'postgres')
    DB_PASSWORD = os.getenv('DB_PASSWORD', '')
    
    @property
    def DATABASE_URL(self):
        return f"postgresql://{self.DB_USER}:{self.DB_PASSWORD}@{self.DB_HOST}:{self.DB_PORT}/{self.DB_NAME}"
    
    # Redis
    REDIS_HOST = os.getenv('REDIS_HOST', 'localhost')
    REDIS_PORT = int(os.getenv('REDIS_PORT', '6379'))
    REDIS_DB = int(os.getenv('REDIS_DB', '0'))
    REDIS_PASSWORD = os.getenv('REDIS_PASSWORD', '')
    
    @property
    def REDIS_URL(self):
        if self.REDIS_PASSWORD:
            return f"redis://:{self.REDIS_PASSWORD}@{self.REDIS_HOST}:{self.REDIS_PORT}/{self.REDIS_DB}"
        return f"redis://{self.REDIS_HOST}:{self.REDIS_PORT}/{self.REDIS_DB}"
    
    # Логирование
    LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO')
    LOG_FILE = os.getenv('LOG_FILE', str(LOG_DIR / 'app.log'))
    
    # API
    API_VERSION = os.getenv('API_VERSION', 'v1')
    ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', 'localhost,127.0.0.1').split(',')
    CORS_ORIGINS = os.getenv('CORS_ORIGINS', '*').split(',')
    
    @classmethod
    def validate(cls):
        """Проверка конфигурации."""
        if cls.APP_ENV == 'production':
            assert cls.SECRET_KEY != 'dev-secret-key-change-me', "SECRET_KEY must be changed in production"
            assert cls.DB_PASSWORD, "DB_PASSWORD is required in production"
            assert cls.REDIS_PASSWORD, "REDIS_PASSWORD is required in production"


class DevelopmentConfig(Config):
    DEBUG = True
    LOG_LEVEL = 'DEBUG'


class ProductionConfig(Config):
    DEBUG = False
    LOG_LEVEL = 'WARNING'


class TestingConfig(Config):
    DEBUG = True
    DB_NAME = 'myapp_test'
    LOG_LEVEL = 'ERROR'


# Выбор конфигурации
def get_config():
    env = os.getenv('APP_ENV', 'development')
    if env == 'production':
        return ProductionConfig
    elif env == 'testing':
        return TestingConfig
    else:
        return DevelopmentConfig


config = get_config()
config.validate()
6.2. Docker-совместимая конфигурация
python
# docker_config.py
import os
from dotenv import load_dotenv

# Загрузка .env (для локальной разработки)
load_dotenv()

# В Docker переменные передаются через -e или env_file


class DockerConfig:
    """Конфигурация для Docker-окружения."""
    
    # Основные настройки
    APP_NAME = os.getenv('APP_NAME', 'myapp')
    APP_ENV = os.getenv('APP_ENV', 'production')
    DEBUG = os.getenv('DEBUG', 'false').lower() == 'true'
    
    # Сервер (для Docker важно слушать 0.0.0.0)
    HOST = os.getenv('HOST', '0.0.0.0')
    PORT = int(os.getenv('PORT', '8000'))
    
    # База данных (подключение по имени сервиса Docker)
    DB_HOST = os.getenv('DB_HOST', 'postgres')
    DB_PORT = int(os.getenv('DB_PORT', '5432'))
    DB_NAME = os.getenv('DB_NAME', 'myapp')
    DB_USER = os.getenv('DB_USER', 'postgres')
    DB_PASSWORD = os.getenv('DB_PASSWORD', '')
    
    @property
    def DATABASE_URL(self):
        return f"postgresql://{self.DB_USER}:{self.DB_PASSWORD}@{self.DB_HOST}:{self.DB_PORT}/{self.DB_NAME}"
    
    # Redis
    REDIS_HOST = os.getenv('REDIS_HOST', 'redis')
    REDIS_PORT = int(os.getenv('REDIS_PORT', '6379'))
    
    @property
    def REDIS_URL(self):
        return f"redis://{self.REDIS_HOST}:{self.REDIS_PORT}/0"
    
    # Логирование (в stdout для Docker)
    LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO')
    LOG_FORMAT = os.getenv('LOG_FORMAT', 'json')


config = DockerConfig()
6.3. docker-compose.yml с переменными окружения
yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    environment:
      - APP_ENV=production
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=myapp
      - DB_USER=postgres
      - DB_PASSWORD=${DB_PASSWORD}
      - REDIS_HOST=redis
      - SECRET_KEY=${SECRET_KEY}
    env_file:
      - .env.prod
    ports:
      - "8000:8000"
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}

volumes:
  postgres_data:
7. Контрольные вопросы
Почему нельзя хранить пароли и ключи в коде?

Как получить переменную окружения в Python?

В чём разница между os.getenv() и os.environ[]?

Что делает библиотека python-dotenv?

Как загрузить разные .env файлы для разных окружений?

Что должно быть в .gitignore для .env файлов?

Зачем нужен файл .env.example?

Как типизировать переменные окружения (числа, булевы, списки)?

Как проверить конфигурацию при запуске приложения?

Как генерировать безопасные секретные ключи?

8. Практическое задание
Задание 1 (базовое)
Создайте .env файл с настройками приложения (DEBUG, DB_HOST, API_KEY). Напишите код, который загружает эти настройки и выводит их.

Задание 2 (среднее)
Создайте класс Settings на основе Pydantic или dataclass с валидацией. Добавьте поддержку разных окружений (dev, prod, test).

Задание 3 (сложное)
Разработайте систему конфигурации для веб-приложения с поддержкой:

Переменных окружения

.env файлов

Файлов конфигурации (YAML/JSON)

Валидации типов

Проверки обязательных переменных

9. Шпаргалка
python
# === БАЗОВОЕ ИСПОЛЬЗОВАНИЕ ===
import os
from dotenv import load_dotenv

load_dotenv()
db_host = os.getenv('DB_HOST', 'localhost')
debug = os.getenv('DEBUG', 'false').lower() == 'true'

# === .env ФАЙЛ ===
APP_NAME=MyApp
DEBUG=true
DB_HOST=localhost
DB_PORT=5432

# === РАЗНЫЕ ОКРУЖЕНИЯ ===
load_dotenv('.env.dev')      # разработка
load_dotenv('.env.prod')     # продакшен

# === ТИПИЗАЦИЯ ===
def get_bool(key: str, default: bool = False) -> bool:
    return os.getenv(key, str(default)).lower() in ('true', '1', 'yes')

def get_list(key: str, separator: str = ',') -> list:
    return os.getenv(key, '').split(separator)

# === ПРОВЕРКА ===
required_vars = ['SECRET_KEY', 'DB_PASSWORD']
missing = [v for v in required_vars if not os.getenv(v)]
if missing:
    raise ValueError(f"Missing: {missing}")

# === ГЕНЕРАЦИЯ КЛЮЧЕЙ ===
import secrets
secret = secrets.token_urlsafe(32)
Итог лекции
Вы сегодня:

✅ Изучили важность отделения конфигурации от кода

✅ Освоили работу с переменными окружения через os.environ

✅ Научились использовать python-dotenv для загрузки .env файлов

✅ Создали типизированные конфигурации с валидацией

✅ Изучили лучшие практики безопасности

✅ Реализовали конфигурации для разных окружений и Docker

Теперь ваши приложения готовы к безопасному деплою в любую среду!
