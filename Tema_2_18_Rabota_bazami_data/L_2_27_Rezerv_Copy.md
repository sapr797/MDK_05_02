# Тема 2.27. Резервное копирование БД (SQLite, PostgreSQL): стратегии, автоматизация

**Цель лекции:**  
Изучить стратегии резервного копирования баз данных, освоить методы создания резервных копий для SQLite и PostgreSQL, научиться автоматизировать процесс бэкапа с помощью Python-скриптов.

> Главная мысль: **Резервная копия — это страховка от потери данных. Регулярные бэкапы — признак профессионального подхода к разработке.**

---

## Содержание

1. [Введение в резервное копирование](#1-введение-в-резервное-копирование)
2. [Резервное копирование SQLite](#2-резервное-копирование-sqlite)
3. [Резервное копирование PostgreSQL](#3-резервное-копирование-postgresql)
4. [Стратегии бэкапа](#4-стратегии-бэкапа)
5. [Автоматизация с Python](#5-автоматизация-с-python)
6. [Восстановление из бэкапа](#6-восстановление-из-бэкапа)
7. [Мониторинг и оповещение](#7-мониторинг-и-оповещение)
8. [Контрольные вопросы](#8-контрольные-вопросы)
9. [Практическое задание](#9-практическое-задание)
10. [Шпаргалка](#10-шпаргалка)

---

## 1. Введение в резервное копирование

### 1.1. Что такое резервное копирование

**Резервное копирование (бэкап)** — это процесс создания копии данных для восстановления в случае их утраты или повреждения.

### 1.2. Зачем нужны бэкапы

| Сценарий | Почему нужен бэкап |
|----------|---------------------|
| Случайное удаление данных | Восстановить потерянную информацию |
| Ошибка в приложении | Откатиться до рабочей версии |
| Атака злоумышленников | Восстановить данные после взлома |
| Сбой оборудования | Не потерять данные при поломке диска |
| Обновление БД | Откатить неудачное обновление |
| Тестирование | Восстановить тестовую среду |

### 1.3. Типы резервного копирования

| Тип | Описание | Размер | Скорость | Время восстановления |
|-----|----------|--------|----------|----------------------|
| **Полный (Full)** | Копия всей БД | Большой | Медленная | Быстрое |
| **Дифференциальный (Differential)** | Изменения с последнего полного | Средний | Быстрая | Среднее |
| **Инкрементный (Incremental)** | Изменения с последнего бэкапа | Маленький | Очень быстрая | Медленное |

### 1.4. Стратегии хранения бэкапов
┌─────────────────────────────────────────────────────────────────┐
│ ПИРАМИДА ХРАНЕНИЯ │
├─────────────────────────────────────────────────────────────────┤
│ │
│ ┌─────────────┐ │
│ │ daily/ │ Ежедневные бэкапы │
│ │ hourly │ (хранятся 7 дней) │
│ ┌─┴─────────────┴─┐ │
│ │ weekly │ Еженедельные бэкапы │
│ │ │ (хранятся 4 недели) │
│ ┌─┴─────────────────┴─┐ │
│ │ monthly │ Ежемесячные бэкапы │
│ │ │ (хранятся 12 месяцев) │
│ ┌─┴─────────────────────┴─┐ │
│ │ yearly │ Годовые бэкапы │
│ │ │ (хранятся годами) │
│ └────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────┘

text

---

## 2. Резервное копирование SQLite

### 2.1. Простое копирование файла

```python
import shutil
from pathlib import Path
from datetime import datetime

def backup_sqlite_simple(db_path: str, backup_dir: str = "backups"):
    """Простое копирование файла SQLite."""
    
    db_file = Path(db_path)
    backup_dir = Path(backup_dir)
    
    # Создаём папку для бэкапов
    backup_dir.mkdir(exist_ok=True)
    
    # Имя бэкапа с временной меткой
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_name = f"{db_file.stem}_{timestamp}.db"
    backup_path = backup_dir / backup_name
    
    # Копируем файл
    shutil.copy2(db_file, backup_path)
    
    print(f"✅ Бэкап создан: {backup_path}")
    return backup_path

# Использование
backup_sqlite_simple("myapp.db", "backups")
2.2. Бэкап с использованием SQLite API (без блокировки)
python
import sqlite3
from pathlib import Path
from datetime import datetime

def backup_sqlite_api(db_path: str, backup_dir: str = "backups"):
    """
    Резервное копирование SQLite с использованием встроенного API.
    Безопасно для работающей БД.
    """
    
    db_file = Path(db_path)
    backup_dir = Path(backup_dir)
    backup_dir.mkdir(exist_ok=True)
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_path = backup_dir / f"{db_file.stem}_{timestamp}.db"
    
    # Подключаемся к исходной БД
    source = sqlite3.connect(db_path)
    
    # Создаём новую БД для бэкапа
    destination = sqlite3.connect(str(backup_path))
    
    # Копируем данные
    with destination:
        source.backup(destination, pages=1, progress=None)
    
    # Закрываем соединения
    source.close()
    destination.close()
    
    print(f"✅ Бэкап создан: {backup_path} (размер: {backup_path.stat().st_size / 1024:.1f} KB)")
    return backup_path

# Использование
backup_sqlite_api("myapp.db", "backups")
2.3. Бэкап с проверкой целостности
python
import sqlite3
import hashlib
from pathlib import Path
from datetime import datetime
import json

class SQLiteBackupManager:
    """Менеджер резервного копирования SQLite."""
    
    def __init__(self, db_path: str, backup_dir: str = "backups"):
        self.db_path = Path(db_path)
        self.backup_dir = Path(backup_dir)
        self.backup_dir.mkdir(exist_ok=True)
        self.metadata_file = self.backup_dir / "backup_metadata.json"
    
    def _calculate_hash(self, file_path: Path) -> str:
        """Вычисляет SHA-256 хеш файла."""
        sha256 = hashlib.sha256()
        with open(file_path, 'rb') as f:
            for chunk in iter(lambda: f.read(4096), b''):
                sha256.update(chunk)
        return sha256.hexdigest()
    
    def _check_integrity(self, db_path: Path) -> bool:
        """Проверяет целостность базы данных."""
        try:
            conn = sqlite3.connect(db_path)
            cursor = conn.cursor()
            cursor.execute("PRAGMA integrity_check")
            result = cursor.fetchone()[0]
            conn.close()
            return result == "ok"
        except Exception as e:
            print(f"Ошибка проверки целостности: {e}")
            return False
    
    def create_backup(self) -> dict:
        """Создаёт резервную копию с метаданными."""
        
        # Проверка исходной БД
        if not self.db_path.exists():
            raise FileNotFoundError(f"База данных не найдена: {self.db_path}")
        
        if not self._check_integrity(self.db_path):
            raise Exception("Исходная база данных повреждена")
        
        # Создание бэкапа
        timestamp = datetime.now()
        backup_name = f"{self.db_path.stem}_{timestamp.strftime('%Y%m%d_%H%M%S')}.db"
        backup_path = self.backup_dir / backup_name
        
        # Копирование с проверкой
        conn = sqlite3.connect(self.db_path)
        conn.backup(sqlite3.connect(backup_path))
        conn.close()
        
        # Проверка бэкапа
        if not self._check_integrity(backup_path):
            backup_path.unlink()
            raise Exception("Созданный бэкап повреждён")
        
        # Сохранение метаданных
        metadata = {
            "backup_name": backup_name,
            "backup_path": str(backup_path),
            "timestamp": timestamp.isoformat(),
            "original_size": self.db_path.stat().st_size,
            "backup_size": backup_path.stat().st_size,
            "hash": self._calculate_hash(backup_path),
            "integrity": "ok"
        }
        
        self._save_metadata(metadata)
        
        print(f"✅ Бэкап создан: {backup_path}")
        print(f"   Размер: {metadata['backup_size'] / 1024:.1f} KB")
        print(f"   Хеш: {metadata['hash'][:16]}...")
        
        return metadata
    
    def _save_metadata(self, metadata: dict):
        """Сохраняет метаданные бэкапа."""
        backups = self.list_backups()
        backups.append(metadata)
        
        with open(self.metadata_file, 'w', encoding='utf-8') as f:
            json.dump(backups, f, indent=2, ensure_ascii=False)
    
    def list_backups(self) -> list:
        """Возвращает список всех бэкапов."""
        if self.metadata_file.exists():
            with open(self.metadata_file, 'r', encoding='utf-8') as f:
                return json.load(f)
        return []
    
    def clean_old_backups(self, keep_days: int = 30):
        """Удаляет старые бэкапы."""
        cutoff = datetime.now().timestamp() - (keep_days * 24 * 3600)
        removed = []
        
        for backup in self.list_backups():
            backup_time = datetime.fromisoformat(backup['timestamp']).timestamp()
            if backup_time < cutoff:
                backup_path = Path(backup['backup_path'])
                if backup_path.exists():
                    backup_path.unlink()
                    removed.append(backup['backup_name'])
        
        # Обновляем метаданные
        backups = [b for b in self.list_backups() 
                   if b['backup_name'] not in removed]
        
        with open(self.metadata_file, 'w', encoding='utf-8') as f:
            json.dump(backups, f, indent=2, ensure_ascii=False)
        
        print(f"🧹 Удалено старых бэкапов: {len(removed)}")
        return removed

# Использование
backup_manager = SQLiteBackupManager("myapp.db", "backups")
backup_manager.create_backup()
backup_manager.clean_old_backups(keep_days=7)
3. Резервное копирование PostgreSQL
3.1. Бэкап через pg_dump
python
import subprocess
import os
from pathlib import Path
from datetime import datetime
from typing import Optional

def postgres_backup_pgdump(
    host: str,
    port: int,
    database: str,
    user: str,
    password: str,
    backup_dir: str = "backups"
) -> Optional[Path]:
    """
    Резервное копирование PostgreSQL с помощью pg_dump.
    """
    
    backup_dir = Path(backup_dir)
    backup_dir.mkdir(exist_ok=True)
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_file = backup_dir / f"{database}_{timestamp}.sql"
    
    # Устанавливаем пароль через переменную окружения
    env = os.environ.copy()
    env["PGPASSWORD"] = password
    
    # Команда pg_dump
    cmd = [
        "pg_dump",
        "-h", host,
        "-p", str(port),
        "-U", user,
        "-d", database,
        "-f", str(backup_file),
        "--format=plain",
        "--verbose"
    ]
    
    try:
        result = subprocess.run(
            cmd,
            env=env,
            capture_output=True,
            text=True,
            check=True
        )
        
        print(f"✅ Бэкап PostgreSQL создан: {backup_file}")
        print(f"   Размер: {backup_file.stat().st_size / 1024:.1f} KB")
        return backup_file
        
    except subprocess.CalledProcessError as e:
        print(f"❌ Ошибка бэкапа: {e}")
        print(f"   stderr: {e.stderr}")
        return None

# Использование
postgres_backup_pgdump(
    host="localhost",
    port=5432,
    database="myapp",
    user="postgres",
    password="secret"
)
3.2. Сжатый бэкап PostgreSQL
python
def postgres_backup_compressed(
    host: str,
    port: int,
    database: str,
    user: str,
    password: str,
    backup_dir: str = "backups"
) -> Optional[Path]:
    """
    Сжатый бэкап PostgreSQL в формате custom.
    """
    
    backup_dir = Path(backup_dir)
    backup_dir.mkdir(exist_ok=True)
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_file = backup_dir / f"{database}_{timestamp}.dump"
    
    env = os.environ.copy()
    env["PGPASSWORD"] = password
    
    # Формат custom: сжатие, можно восстановить частично
    cmd = [
        "pg_dump",
        "-h", host,
        "-p", str(port),
        "-U", user,
        "-d", database,
        "-f", str(backup_file),
        "--format=custom",
        "--compress=9",
        "--verbose"
    ]
    
    try:
        result = subprocess.run(cmd, env=env, capture_output=True, text=True, check=True)
        
        print(f"✅ Сжатый бэкап PostgreSQL создан: {backup_file}")
        size_mb = backup_file.stat().st_size / (1024 * 1024)
        print(f"   Размер: {size_mb:.2f} MB")
        return backup_file
        
    except subprocess.CalledProcessError as e:
        print(f"❌ Ошибка бэкапа: {e}")
        return None
3.3. Бэкап только схемы или только данных
python
def postgres_backup_schema_only(
    host: str, port: int, database: str, user: str, password: str, backup_dir: str = "backups"
) -> Optional[Path]:
    """Бэкап только схемы БД (без данных)."""
    
    backup_dir = Path(backup_dir)
    backup_dir.mkdir(exist_ok=True)
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_file = backup_dir / f"{database}_schema_{timestamp}.sql"
    
    env = os.environ.copy()
    env["PGPASSWORD"] = password
    
    cmd = [
        "pg_dump",
        "-h", host, "-p", str(port), "-U", user,
        "-d", database, "-f", str(backup_file),
        "--schema-only", "--format=plain"
    ]
    
    subprocess.run(cmd, env=env, check=True)
    print(f"✅ Бэкап схемы: {backup_file}")
    return backup_file


def postgres_backup_data_only(
    host: str, port: int, database: str, user: str, password: str, backup_dir: str = "backups"
) -> Optional[Path]:
    """Бэкап только данных (без схемы)."""
    
    backup_dir = Path(backup_dir)
    backup_dir.mkdir(exist_ok=True)
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_file = backup_dir / f"{database}_data_{timestamp}.sql"
    
    env = os.environ.copy()
    env["PGPASSWORD"] = password
    
    cmd = [
        "pg_dump",
        "-h", host, "-p", str(port), "-U", user,
        "-d", database, "-f", str(backup_file),
        "--data-only", "--format=plain", "--inserts"
    ]
    
    subprocess.run(cmd, env=env, check=True)
    print(f"✅ Бэкап данных: {backup_file}")
    return backup_file
4. Стратегии бэкапа
4.1. Полная стратегия (Full + Incremental)
python
class BackupStrategy:
    """Стратегии резервного копирования."""
    
    def __init__(self, backup_dir: str = "backups"):
        self.backup_dir = Path(backup_dir)
        self.backup_dir.mkdir(exist_ok=True)
        self.full_backup_dir = self.backup_dir / "full"
        self.inc_backup_dir = self.backup_dir / "incremental"
        self.full_backup_dir.mkdir(exist_ok=True)
        self.inc_backup_dir.mkdir(exist_ok=True)
    
    def full_backup_sqlite(self, db_path: str) -> Path:
        """Полный бэкап SQLite."""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_name = f"full_{timestamp}.db"
        backup_path = self.full_backup_dir / backup_name
        
        import shutil
        shutil.copy2(db_path, backup_path)
        
        print(f"📦 Полный бэкап: {backup_path}")
        return backup_path
    
    def incremental_backup_sqlite(self, db_path: str, last_full: Path) -> Path:
        """Инкрементный бэкап (копирование WAL файла)."""
        import shutil
        
        wal_path = Path(db_path).parent / f"{Path(db_path).stem}.wal"
        if not wal_path.exists():
            print("⚠️ Нет изменений для инкрементного бэкапа")
            return None
        
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_name = f"inc_{timestamp}.wal"
        backup_path = self.inc_backup_dir / backup_name
        
        shutil.copy2(wal_path, backup_path)
        
        print(f"📈 Инкрементный бэкап: {backup_path}")
        return backup_path
    
    def rotate_backups(self, keep_full: int = 7, keep_inc: int = 30):
        """Ротация бэкапов (удаление старых)."""
        now = datetime.now()
        
        # Удаление старых полных бэкапов
        for backup in self.full_backup_dir.glob("full_*.db"):
            # Извлекаем дату из имени
            date_str = backup.stem.replace("full_", "")
            backup_date = datetime.strptime(date_str, "%Y%m%d_%H%M%S")
            
            if (now - backup_date).days > keep_full:
                backup.unlink()
                print(f"🗑️ Удалён старый полный бэкап: {backup.name}")
        
        # Удаление старых инкрементных бэкапов
        for backup in self.inc_backup_dir.glob("inc_*.wal"):
            date_str = backup.stem.replace("inc_", "")
            backup_date = datetime.strptime(date_str, "%Y%m%d_%H%M%S")
            
            if (now - backup_date).days > keep_inc:
                backup.unlink()
                print(f"🗑️ Удалён старый инкрементный бэкап: {backup.name}")
4.2. Стратегия 3-2-1 (Golden Backup Rule)
text
┌─────────────────────────────────────────────────────────────────┐
│                    ПРАВИЛО 3-2-1                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  3 копии данных                                                 │
│  ├── 1 основная (рабочая БД)                                   │
│  ├── 2 резервная (локальный бэкап)                             │
│  └── 3 архивная (удалённое хранилище)                          │
│                                                                 │
│  2 разных носителя                                              │
│  ├── Локальный диск                                            │
│  └── Внешний диск / Сеть / Облако                              │
│                                                                 │
│  1 офлайн-копия (вне основного места)                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
5. Автоматизация с Python
5.1. Полный скрипт автоматического бэкапа
python
#!/usr/bin/env python3
"""
backup_manager.py — Полный менеджер резервного копирования.
"""

import os
import sys
import json
import shutil
import sqlite3
import subprocess
from pathlib import Path
from datetime import datetime
from typing import Dict, List, Optional
import logging

# Настройка логирования
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('backup.log'),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger(__name__)


class BackupManager:
    """Универсальный менеджер резервного копирования."""
    
    def __init__(self, config_file: str = "backup_config.json"):
        self.config = self._load_config(config_file)
        self.backup_dir = Path(self.config.get("backup_dir", "backups"))
        self.backup_dir.mkdir(exist_ok=True)
    
    def _load_config(self, config_file: str) -> dict:
        """Загружает конфигурацию из JSON."""
        default_config = {
            "backup_dir": "backups",
            "databases": {
                "sqlite": [
                    {"name": "myapp", "path": "myapp.db", "enabled": True}
                ],
                "postgresql": [
                    {
                        "name": "myapp_pg",
                        "host": "localhost",
                        "port": 5432,
                        "database": "myapp",
                        "user": "postgres",
                        "password": "password",
                        "enabled": False
                    }
                ]
            },
            "retention": {
                "daily": 7,
                "weekly": 4,
                "monthly": 12
            },
            "remote": {
                "enabled": False,
                "type": "s3",  # s3, ftp, ssh
                "bucket": "my-backups",
                "path": "/backups/"
            }
        }
        
        if Path(config_file).exists():
            with open(config_file, 'r', encoding='utf-8') as f:
                config = json.load(f)
                # Объединение с дефолтными настройками
                for key in default_config:
                    if key not in config:
                        config[key] = default_config[key]
                return config
        else:
            with open(config_file, 'w', encoding='utf-8') as f:
                json.dump(default_config, f, indent=2, ensure_ascii=False)
            logger.info(f"Создан конфиг: {config_file}")
            return default_config
    
    def backup_sqlite(self, db_config: dict) -> Optional[Path]:
        """Бэкап SQLite базы данных."""
        db_path = Path(db_config['path'])
        
        if not db_path.exists():
            logger.error(f"База данных не найдена: {db_path}")
            return None
        
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_name = f"{db_config['name']}_{timestamp}.db"
        backup_path = self.backup_dir / backup_name
        
        try:
            # Безопасное копирование через SQLite API
            source = sqlite3.connect(db_path)
            dest = sqlite3.connect(backup_path)
            source.backup(dest)
            source.close()
            dest.close()
            
            logger.info(f"✅ SQLite бэкап: {backup_path} ({backup_path.stat().st_size / 1024:.1f} KB)")
            return backup_path
            
        except Exception as e:
            logger.error(f"❌ Ошибка бэкапа SQLite: {e}")
            return None
    
    def backup_postgresql(self, db_config: dict) -> Optional[Path]:
        """Бэкап PostgreSQL базы данных."""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_name = f"{db_config['name']}_{timestamp}.dump"
        backup_path = self.backup_dir / backup_name
        
        env = os.environ.copy()
        env["PGPASSWORD"] = db_config['password']
        
        cmd = [
            "pg_dump",
            "-h", db_config['host'],
            "-p", str(db_config['port']),
            "-U", db_config['user'],
            "-d", db_config['database'],
            "-f", str(backup_path),
            "--format=custom",
            "--compress=6"
        ]
        
        try:
            result = subprocess.run(cmd, env=env, capture_output=True, text=True)
            
            if result.returncode == 0:
                size_mb = backup_path.stat().st_size / (1024 * 1024)
                logger.info(f"✅ PostgreSQL бэкап: {backup_path} ({size_mb:.2f} MB)")
                return backup_path
            else:
                logger.error(f"❌ Ошибка pg_dump: {result.stderr}")
                return None
                
        except FileNotFoundError:
            logger.error("❌ pg_dump не найден. Установите PostgreSQL client.")
            return None
        except Exception as e:
            logger.error(f"❌ Ошибка бэкапа PostgreSQL: {e}")
            return None
    
    def run_backups(self) -> Dict[str, List[Path]]:
        """Запускает бэкапы для всех настроенных БД."""
        results = {"sqlite": [], "postgresql": []}
        
        logger.info("🚀 Запуск резервного копирования...")
        
        # SQLite
        for db_config in self.config["databases"]["sqlite"]:
            if db_config.get("enabled", True):
                backup = self.backup_sqlite(db_config)
                if backup:
                    results["sqlite"].append(backup)
        
        # PostgreSQL
        for db_config in self.config["databases"]["postgresql"]:
            if db_config.get("enabled", True):
                backup = self.backup_postgresql(db_config)
                if backup:
                    results["postgresql"].append(backup)
        
        # Очистка старых бэкапов
        self.cleanup_old_backups()
        
        # Синхронизация с удалённым хранилищем
        if self.config["remote"]["enabled"]:
            self.sync_to_remote()
        
        logger.info(f"✅ Бэкапы завершены. SQLite: {len(results['sqlite'])}, PostgreSQL: {len(results['postgresql'])}")
        return results
    
    def cleanup_old_backups(self):
        """Удаляет старые бэкапы в соответствии с политикой хранения."""
        now = datetime.now()
        retention = self.config["retention"]
        
        for backup_file in self.backup_dir.glob("*"):
            if not backup_file.is_file():
                continue
            
            # Извлекаем дату из имени файла
            try:
                name = backup_file.stem
                date_str = name.split('_')[-1] if '_' in name else name
                backup_date = datetime.strptime(date_str, "%Y%m%d_%H%M%S")
                age_days = (now - backup_date).days
            except (ValueError, IndexError):
                continue
            
            # Ежедневные бэкапы старше 7 дней
            if age_days > retention["daily"]:
                # Проверяем, является ли бэкап еженедельным
                if backup_date.weekday() != 0:  # не понедельник
                    backup_file.unlink()
                    logger.info(f"🗑️ Удалён старый бэкап: {backup_file.name}")
                elif age_days > retention["weekly"] * 7:
                    # Старые еженедельные бэкапы
                    if backup_date.day > 7:  # не первая неделя месяца
                        backup_file.unlink()
                        logger.info(f"🗑️ Удалён старый еженедельный бэкап: {backup_file.name}")
    
    def sync_to_remote(self):
        """Синхронизация с удалённым хранилищем."""
        remote_config = self.config["remote"]
        
        if remote_config["type"] == "s3":
            self._sync_to_s3()
        elif remote_config["type"] == "ftp":
            self._sync_to_ftp()
        elif remote_config["type"] == "ssh":
            self._sync_to_ssh()
    
    def _sync_to_s3(self):
        """Синхронизация с AWS S3."""
        try:
            import boto3
            from botocore.exceptions import ClientError
            
            s3 = boto3.client('s3')
            bucket = self.config["remote"]["bucket"]
            prefix = self.config["remote"].get("path", "backups/")
            
            for backup_file in self.backup_dir.glob("*"):
                if backup_file.is_file():
                    key = f"{prefix}{backup_file.name}"
                    s3.upload_file(str(backup_file), bucket, key)
                    logger.info(f"☁️ Загружено в S3: {key}")
                    
        except ImportError:
            logger.warning("boto3 не установлен. Пропуск синхронизации с S3.")
        except Exception as e:
            logger.error(f"Ошибка синхронизации с S3: {e}")


# ============================================================
# ЗАПУСК
# ============================================================

def main():
    """Главная функция для запуска бэкапа из командной строки."""
    import argparse
    
    parser = argparse.ArgumentParser(description="Менеджер резервного копирования БД")
    parser.add_argument("--config", default="backup_config.json", help="Файл конфигурации")
    parser.add_argument("--backup-only", action="store_true", help="Только бэкап")
    parser.add_argument("--cleanup-only", action="store_true", help="Только очистка")
    
    args = parser.parse_args()
    
    manager = BackupManager(args.config)
    
    if args.cleanup_only:
        manager.cleanup_old_backups()
    elif args.backup_only:
        manager.run_backups()
    else:
        # Полный цикл
        manager.run_backups()


if __name__ == "__main__":
    main()
5.2. Настройка cron для автоматического запуска
bash
# Редактирование crontab
crontab -e

# Запуск бэкапа каждый день в 2 часа ночи
0 2 * * * /usr/bin/python3 /path/to/backup_manager.py --config /path/to/backup_config.json >> /var/log/backup.log 2>&1

# Запуск бэкапа каждый час для важных БД
0 * * * * /usr/bin/python3 /path/to/backup_manager.py --backup-only

# Очистка старых бэкапов раз в неделю
0 3 * * 0 /usr/bin/python3 /path/to/backup_manager.py --cleanup-only
5.3. Windows Task Scheduler (через Python)
python
# setup_scheduler.py
import subprocess
import sys

def create_windows_task():
    """Создаёт задачу в планировщике Windows."""
    
    task_name = "DatabaseBackup"
    python_path = sys.executable
    script_path = Path(__file__).parent / "backup_manager.py"
    
    cmd = [
        "schtasks", "/create", "/tn", task_name,
        "/tr", f"\"{python_path} {script_path}\"",
        "/sc", "daily", "/st", "02:00",
        "/ru", "SYSTEM"
    ]
    
    subprocess.run(cmd, shell=True)
    print(f"✅ Задача '{task_name}' создана")

if __name__ == "__main__":
    create_windows_task()
6. Восстановление из бэкапа
6.1. Восстановление SQLite
python
def restore_sqlite(backup_path: str, target_path: str) -> bool:
    """Восстановление SQLite из бэкапа."""
    
    backup_path = Path(backup_path)
    target_path = Path(target_path)
    
    if not backup_path.exists():
        print(f"❌ Бэкап не найден: {backup_path}")
        return False
    
    try:
        # Проверка целостности бэкапа
        conn = sqlite3.connect(backup_path)
        cursor = conn.cursor()
        cursor.execute("PRAGMA integrity_check")
        result = cursor.fetchone()[0]
        conn.close()
        
        if result != "ok":
            print("❌ Бэкап повреждён")
            return False
        
        # Создание резервной копии текущей БД (на всякий случай)
        if target_path.exists():
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            current_backup = target_path.with_suffix(f".{timestamp}.backup")
            import shutil
            shutil.copy2(target_path, current_backup)
            print(f"📦 Текущая БД сохранена как: {current_backup}")
        
        # Восстановление
        import shutil
        shutil.copy2(backup_path, target_path)
        
        print(f"✅ База данных восстановлена из: {backup_path}")
        return True
        
    except Exception as e:
        print(f"❌ Ошибка восстановления: {e}")
        return False
6.2. Восстановление PostgreSQL
python
def restore_postgresql(
    backup_path: str,
    host: str, port: int, database: str, user: str, password: str
) -> bool:
    """Восстановление PostgreSQL из бэкапа."""
    
    backup_path = Path(backup_path)
    
    if not backup_path.exists():
        print(f"❌ Бэкап не найден: {backup_path}")
        return False
    
    env = os.environ.copy()
    env["PGPASSWORD"] = password
    
    # Сначала убиваем существующие соединения
    kill_cmd = [
        "psql", "-h", host, "-p", str(port), "-U", user,
        "-d", "postgres", "-c",
        f"SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = '{database}';"
    ]
    subprocess.run(kill_cmd, env=env, capture_output=True)
    
    # Удаляем старую БД
    drop_cmd = ["dropdb", "-h", host, "-p", str(port), "-U", user, "--if-exists", database]
    subprocess.run(drop_cmd, env=env, capture_output=True)
    
    # Создаём новую БД
    create_cmd = ["createdb", "-h", host, "-p", str(port), "-U", user, database]
    subprocess.run(create_cmd, env=env, capture_output=True)
    
    # Восстанавливаем из бэкапа
    restore_cmd = [
        "pg_restore",
        "-h", host, "-p", str(port), "-U", user,
        "-d", database,
        "--verbose", "--clean", "--if-exists",
        str(backup_path)
    ]
    
    try:
        result = subprocess.run(restore_cmd, env=env, capture_output=True, text=True)
        
        if result.returncode == 0:
            print(f"✅ База данных {database} восстановлена из: {backup_path}")
            return True
        else:
            print(f"❌ Ошибка восстановления: {result.stderr}")
            return False
            
    except Exception as e:
        print(f"❌ Ошибка: {e}")
        return False
7. Мониторинг и оповещение
python
# alerting.py
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from datetime import datetime

class BackupAlert:
    """Система оповещения о бэкапах."""
    
    def __init__(self, smtp_config: dict):
        self.smtp_config = smtp_config
    
    def send_telegram(self, message: str, bot_token: str, chat_id: str):
        """Отправка уведомления в Telegram."""
        import requests
        
        url = f"https://api.telegram.org/bot{bot_token}/sendMessage"
        payload = {
            "chat_id": chat_id,
            "text": message,
            "parse_mode": "HTML"
        }
        
        try:
            response = requests.post(url, json=payload, timeout=10)
            return response.status_code == 200
        except Exception as e:
            print(f"Ошибка отправки в Telegram: {e}")
            return False
    
    def send_email(self, subject: str, body: str, to_emails: list):
        """Отправка уведомления по email."""
        
        msg = MIMEMultipart()
        msg["From"] = self.smtp_config["from"]
        msg["To"] = ", ".join(to_emails)
        msg["Subject"] = subject
        
        msg.attach(MIMEText(body, "html"))
        
        try:
            with smtplib.SMTP(self.smtp_config["host"], self.smtp_config["port"]) as server:
                server.starttls()
                server.login(self.smtp_config["user"], self.smtp_config["password"])
                server.send_message(msg)
            return True
        except Exception as e:
            print(f"Ошибка отправки email: {e}")
            return False
    
    def notify_backup_success(self, backup_results: dict):
        """Уведомление об успешном бэкапе."""
        message = f"""
        ✅ <b>Резервное копирование выполнено успешно</b>
        
        📅 Время: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
        
        📊 Статистика:
        • SQLite бэкапов: {len(backup_results.get('sqlite', []))}
        • PostgreSQL бэкапов: {len(backup_results.get('postgresql', []))}
        
        📁 Папка: backups/
        """
        
        self.send_telegram(message, self.smtp_config["telegram_bot"], self.smtp_config["telegram_chat"])
    
    def notify_backup_failure(self, error: str):
        """Уведомление об ошибке бэкапа."""
        message = f"""
        ❌ <b>ОШИБКА РЕЗЕРВНОГО КОПИРОВАНИЯ</b>
        
        📅 Время: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
        
        🔴 Ошибка: {error}
        
        ⚠️ Требуется ручное вмешательство!
        """
        
        self.send_telegram(message, self.smtp_config["telegram_bot"], self.smtp_config["telegram_chat"])
        self.send_email("Backup Failure", message, self.smtp_config["alert_emails"])
8. Контрольные вопросы
Какие существуют типы резервного копирования?

В чём разница между полным и инкрементным бэкапом?

Как создать бэкап SQLite без блокировки БД?

Какая команда используется для бэкапа PostgreSQL?

Что такое правило 3-2-1?

Как настроить автоматический бэкап через cron?

Как проверить целостность бэкапа SQLite?

Как восстановить PostgreSQL из бэкапа?

Какие данные нужно включать в метаданные бэкапа?

Как организовать ротацию бэкапов?

9. Практическое задание
Задание 1 (базовое)
Напишите скрипт, который создаёт резервную копию SQLite базы данных с временной меткой в имени файла.

Задание 2 (среднее)
Создайте систему автоматического бэкапа PostgreSQL с ротацией (хранить 7 последних бэкапов).

Задание 3 (сложное)
Разработайте полноценную систему резервного копирования с:

Поддержкой SQLite и PostgreSQL

Ежедневным расписанием

Очисткой старых бэкапов

Уведомлениями в Telegram

Проверкой целостности

10. Шпаргалка
bash
# === SQLite ===
# Копирование файла
cp database.db backup_$(date +%Y%m%d).db

# Через sqlite3 shell
sqlite3 database.db ".backup backup.db"

# Через Python API
conn = sqlite3.connect('database.db')
conn.backup(sqlite3.connect('backup.db'))

# === PostgreSQL ===
# Бэкап
pg_dump -h localhost -U user -d database > backup.sql
pg_dump -Fc -f backup.dump database  # custom format with compression

# Восстановление
psql -h localhost -U user -d database < backup.sql
pg_restore -d database backup.dump

# === Автоматизация ===
# crontab
0 2 * * * /path/to/backup_script.sh

# === Ротация ===
find backups/ -name "*.db" -mtime +7 -delete
Итог лекции
Вы сегодня:

✅ Изучили стратегии резервного копирования

✅ Научились создавать бэкапы SQLite и PostgreSQL

✅ Освоили автоматизацию с помощью Python-скриптов

✅ Изучили ротацию и очистку старых бэкапов

✅ Реализовали систему оповещения о результатах

Теперь ваши данные защищены от потери!
