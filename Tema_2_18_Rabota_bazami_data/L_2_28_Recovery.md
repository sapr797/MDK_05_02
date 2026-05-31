# Тема 2.28. Восстановление приложения после сбоя: точки восстановления, миграции

**Цель лекции:**  
Изучить стратегии восстановления приложений после сбоев, освоить создание точек восстановления, интеграцию с миграциями базы данных, научиться реализовывать механизмы отката и восстановления работоспособности системы.

> Главная мысль: **Сбои неизбежны. Профессионал не тот, у кого их нет, а тот, кто готов к их последствиям и умеет быстро восстановиться.**

---

## Содержание

1. [Введение в восстановление приложений](#1-введение-в-восстановление-приложений)
2. [Точки восстановления (Restore Points)](#2-точки-восстановления-restore-points)
3. [Интеграция с миграциями Alembic](#3-интеграция-с-миграциями-alembic)
4. [Стратегии отката изменений](#4-стратегии-отката-изменений)
5. [Комплексное восстановление](#5-комплексное-восстановление)
6. [Автоматизация восстановления](#6-автоматизация-восстановления)
7. [Мониторинг и оповещение о сбоях](#7-мониторинг-и-оповещение-о-сбоях)
8. [Контрольные вопросы](#8-контрольные-вопросы)
9. [Практическое задание](#9-практическое-задание)
10. [Шпаргалка](#10-шпаргалка)

---

## 1. Введение в восстановление приложений

### 1.1. Типы сбоев

| Тип сбоя | Пример | Влияние |
|----------|--------|---------|
| **Аппаратный** | Выход из строя диска | Потеря данных |
| **Программный** | Ошибка в коде, миграция удалила данные | Нарушение логики |
| **Человеческий** | Случайное удаление таблицы | Потеря данных |
| **Сетевой** | Отказ связи, утеря соединения | Временная недоступность |
| **Безопасности** | Взлом, утечка данных | Компрометация системы |

### 1.2. Показатели восстановления
┌─────────────────────────────────────────────────────────────────┐
│ RPO и RTO │
├─────────────────────────────────────────────────────────────────┤
│ │
│ RPO (Recovery Point Objective) │
│ └── Насколько старые данные можно потерять │
│ Пример: 15 минут → теряем максимум 15 минут данных │
│ │
│ RTO (Recovery Time Objective) │
│ └── За какое время система должна быть восстановлена │
│ Пример: 1 час → через час система должна работать │
│ │
└─────────────────────────────────────────────────────────────────┘

text

### 1.3. Пирамида восстановления
┌─────────────────┐
│ Full Recovery │ ← Полная репликация
┌─┴─────────────────┴─┐
│ Point-in-Time │ ← Восстановление на момент времени
┌─┴─────────────────────┴─┐
│ Transaction Logs │ ← Ручное применение WAL
┌─┴─────────────────────────┴─┐
│ Daily Backups │ ← Ежедневные бэкапы
┌─┴─────────────────────────────┴─┐
│ Disaster Recovery │ ← Геораспределение
└─────────────────────────────────┘

text

---

## 2. Точки восстановления (Restore Points)

### 2.1. Создание точек восстановления

```python
# restore_point.py
import json
import shutil
import hashlib
from pathlib import Path
from datetime import datetime
from typing import Dict, List, Optional
import sqlite3


class RestorePointManager:
    """Менеджер точек восстановления приложения."""
    
    def __init__(self, app_dir: str, backup_dir: str = "restore_points"):
        self.app_dir = Path(app_dir)
        self.backup_dir = Path(backup_dir)
        self.backup_dir.mkdir(exist_ok=True)
        self.manifest_file = self.backup_dir / "manifest.json"
    
    def create_restore_point(self, description: str = "") -> Dict:
        """
        Создаёт точку восстановления: копирует все важные файлы.
        
        Args:
            description: Описание точки восстановления
        
        Returns:
            dict: Метаданные точки восстановления
        """
        timestamp = datetime.now()
        point_id = timestamp.strftime("%Y%m%d_%H%M%S")
        point_dir = self.backup_dir / point_id
        point_dir.mkdir(exist_ok=True)
        
        files_copied = []
        
        # Копируем файлы приложения
        for file_path in self.app_dir.rglob("*"):
            if file_path.is_file() and self._should_backup(file_path):
                relative_path = file_path.relative_to(self.app_dir)
                dest_path = point_dir / relative_path
                dest_path.parent.mkdir(parents=True, exist_ok=True)
                
                shutil.copy2(file_path, dest_path)
                files_copied.append(str(relative_path))
        
        # Сохраняем метаданные
        metadata = {
            "id": point_id,
            "timestamp": timestamp.isoformat(),
            "description": description,
            "files_count": len(files_copied),
            "files": files_copied,
            "app_version": self._get_app_version()
        }
        
        # Сохраняем метаданные в папку точки
        with open(point_dir / "metadata.json", "w", encoding='utf-8') as f:
            json.dump(metadata, f, indent=2, ensure_ascii=False)
        
        # Обновляем глобальный манифест
        self._update_manifest(metadata)
        
        print(f"✅ Точка восстановления создана: {point_id}")
        print(f"   Файлов сохранено: {len(files_copied)}")
        
        return metadata
    
    def _should_backup(self, file_path: Path) -> bool:
        """Определяет, нужно ли бэкапить файл."""
        # Исключаем временные файлы
        exclude_patterns = [
            "*.pyc", "__pycache__", "*.log", "*.tmp",
            "venv", ".venv", ".env", ".git"
        ]
        
        for pattern in exclude_patterns:
            if file_path.match(pattern):
                return False
        return True
    
    def _get_app_version(self) -> str:
        """Возвращает версию приложения."""
        version_file = self.app_dir / "version.txt"
        if version_file.exists():
            return version_file.read_text().strip()
        return "unknown"
    
    def _update_manifest(self, metadata: Dict):
        """Обновляет глобальный манифест точек восстановления."""
        manifest = self.list_restore_points()
        manifest.append(metadata)
        
        with open(self.manifest_file, "w", encoding='utf-8') as f:
            json.dump(manifest, f, indent=2, ensure_ascii=False)
    
    def list_restore_points(self) -> List[Dict]:
        """Возвращает список всех точек восстановления."""
        if self.manifest_file.exists():
            with open(self.manifest_file, "r", encoding='utf-8') as f:
                return json.load(f)
        return []
    
    def restore(self, point_id: str, target_dir: Optional[Path] = None) -> bool:
        """
        Восстанавливает приложение из точки восстановления.
        
        Args:
            point_id: ID точки восстановления
            target_dir: Целевая директория (если не указана — исходная)
        """
        point_dir = self.backup_dir / point_id
        
        if not point_dir.exists():
            print(f"❌ Точка восстановления не найдена: {point_id}")
            return False
        
        target = target_dir or self.app_dir
        
        # Загружаем метаданные
        with open(point_dir / "metadata.json", "r", encoding='utf-8') as f:
            metadata = json.load(f)
        
        print(f"📦 Восстановление из точки: {point_id}")
        print(f"   Описание: {metadata['description']}")
        print(f"   Дата: {metadata['timestamp']}")
        
        # Создаём бэкап текущего состояния перед восстановлением
        if target == self.app_dir:
            self._backup_current_state()
        
        # Восстанавливаем файлы
        for file_path in point_dir.rglob("*"):
            if file_path.is_file() and file_path.name != "metadata.json":
                relative_path = file_path.relative_to(point_dir)
                dest_path = target / relative_path
                dest_path.parent.mkdir(parents=True, exist_ok=True)
                shutil.copy2(file_path, dest_path)
        
        print(f"✅ Приложение восстановлено из точки {point_id}")
        return True
    
    def _backup_current_state(self):
        """Создаёт резервную копию текущего состояния перед восстановлением."""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_name = f"pre_restore_{timestamp}"
        backup_path = self.backup_dir / backup_name
        
        shutil.copytree(self.app_dir, backup_path, ignore=shutil.ignore_patterns(
            "__pycache__", "*.pyc", "venv", ".venv", ".git"
        ))
        print(f"📦 Создана резервная копия текущего состояния: {backup_name}")
    
    def delete_restore_point(self, point_id: str):
        """Удаляет точку восстановления."""
        point_dir = self.backup_dir / point_id
        if point_dir.exists():
            shutil.rmtree(point_dir)
            print(f"🗑️ Точка восстановления удалена: {point_id}")
            
            # Обновляем манифест
            manifest = [p for p in self.list_restore_points() if p['id'] != point_id]
            with open(self.manifest_file, "w", encoding='utf-8') as f:
                json.dump(manifest, f, indent=2, ensure_ascii=False)


# Использование
manager = RestorePointManager("/path/to/app")
manager.create_restore_point("Перед обновлением до версии 2.0")
manager.restore("20250115_143025")
2.2. Точки восстановления с состоянием БД
python
class DatabaseRestorePoint:
    """Точки восстановления для базы данных."""
    
    def __init__(self, db_path: str, restore_dir: str = "db_restore_points"):
        self.db_path = Path(db_path)
        self.restore_dir = Path(restore_dir)
        self.restore_dir.mkdir(exist_ok=True)
    
    def create_restore_point(self, description: str = "") -> Dict:
        """Создаёт точку восстановления БД."""
        
        timestamp = datetime.now()
        point_id = timestamp.strftime("%Y%m%d_%H%M%S")
        point_path = self.restore_dir / f"{point_id}.db"
        
        # Безопасное копирование SQLite
        conn = sqlite3.connect(self.db_path)
        backup_conn = sqlite3.connect(point_path)
        conn.backup(backup_conn)
        conn.close()
        backup_conn.close()
        
        # Сохраняем метаданные
        metadata = {
            "id": point_id,
            "timestamp": timestamp.isoformat(),
            "description": description,
            "db_size": point_path.stat().st_size,
            "integrity": self._check_integrity(point_path)
        }
        
        with open(self.restore_dir / f"{point_id}_meta.json", "w", encoding='utf-8') as f:
            json.dump(metadata, f, indent=2)
        
        print(f"✅ Точка восстановления БД: {point_id}")
        return metadata
    
    def _check_integrity(self, db_path: Path) -> bool:
        """Проверка целостности БД."""
        try:
            conn = sqlite3.connect(db_path)
            cursor = conn.cursor()
            cursor.execute("PRAGMA integrity_check")
            result = cursor.fetchone()[0]
            conn.close()
            return result == "ok"
        except:
            return False
    
    def restore(self, point_id: str) -> bool:
        """Восстанавливает БД из точки восстановления."""
        point_path = self.restore_dir / f"{point_id}.db"
        
        if not point_path.exists():
            print(f"❌ Точка восстановления не найдена: {point_id}")
            return False
        
        # Проверка целостности
        if not self._check_integrity(point_path):
            print(f"❌ Точка восстановления повреждена")
            return False
        
        # Создаём бэкап текущей БД
        current_backup = self.db_path.with_suffix(f".{datetime.now().strftime('%Y%m%d_%H%M%S')}.backup")
        shutil.copy2(self.db_path, current_backup)
        print(f"📦 Создан бэкап текущей БД: {current_backup}")
        
        # Восстанавливаем
        shutil.copy2(point_path, self.db_path)
        
        print(f"✅ База данных восстановлена из точки {point_id}")
        return True


db_restore = DatabaseRestorePoint("myapp.db")
db_restore.create_restore_point("Перед миграцией")
db_restore.restore("20250115_143025")
3. Интеграция с миграциями Alembic
3.1. Миграция с точкой восстановления
python
# migrate_with_restore.py
from alembic.config import Config
from alembic import command
from pathlib import Path
from datetime import datetime
import json


class SafeMigrator:
    """Безопасный мигратор с точками восстановления."""
    
    def __init__(self, alembic_cfg_path: str, db_path: str, backup_dir: str = "migration_backups"):
        self.alembic_cfg = Config(alembic_cfg_path)
        self.db_path = Path(db_path)
        self.backup_dir = Path(backup_dir)
        self.backup_dir.mkdir(exist_ok=True)
    
    def backup_before_migration(self, migration_name: str) -> str:
        """Создаёт резервную копию перед миграцией."""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_name = f"{timestamp}_{migration_name}.db"
        backup_path = self.backup_dir / backup_name
        
        # Копирование БД
        conn = sqlite3.connect(self.db_path)
        backup_conn = sqlite3.connect(backup_path)
        conn.backup(backup_conn)
        conn.close()
        backup_conn.close()
        
        # Сохраняем метаданные
        metadata = {
            "name": migration_name,
            "timestamp": timestamp,
            "backup_file": backup_name,
            "alembic_version": self._get_current_revision()
        }
        
        with open(self.backup_dir / f"{timestamp}_metadata.json", "w") as f:
            json.dump(metadata, f, indent=2)
        
        print(f"📦 Создана резервная копия перед миграцией: {backup_name}")
        return backup_name
    
    def _get_current_revision(self) -> str:
        """Получает текущую ревизию Alembic."""
        from alembic.script import ScriptDirectory
        from alembic.runtime.migration import MigrationContext
        
        engine = create_engine(f"sqlite:///{self.db_path}")
        conn = engine.connect()
        context = MigrationContext.configure(conn)
        current = context.get_current_revision()
        conn.close()
        return current
    
    def upgrade(self, revision: str = "head") -> bool:
        """Применяет миграцию с предварительным бэкапом."""
        
        # Получаем информацию о миграции
        migration_info = self._get_migration_info(revision)
        
        # Создаём бэкап
        self.backup_before_migration(migration_info)
        
        try:
            # Применяем миграцию
            command.upgrade(self.alembic_cfg, revision)
            print(f"✅ Миграция {revision} применена успешно")
            return True
            
        except Exception as e:
            print(f"❌ Ошибка при миграции: {e}")
            print(f"⚠️ Требуется ручное восстановление из бэкапа")
            return False
    
    def _get_migration_info(self, revision: str) -> str:
        """Получает информацию о миграции."""
        from alembic.script import ScriptDirectory
        
        script = ScriptDirectory.from_config(self.alembic_cfg)
        if revision == "head":
            # Находим последнюю миграцию
            heads = script.get_heads()
            if heads:
                rev = script.get_revision(heads[0])
                return f"{rev.revision}_{rev.module[:20]}" if rev else "head"
        return revision
    
    def downgrade_with_backup(self, revision: str = "base") -> bool:
        """Откат миграции с предварительным бэкапом."""
        
        # Создаём бэкап перед откатом
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_path = self.backup_dir / f"{timestamp}_pre_downgrade.db"
        shutil.copy2(self.db_path, backup_path)
        
        try:
            command.downgrade(self.alembic_cfg, revision)
            print(f"✅ Откат до {revision} выполнен")
            return True
            
        except Exception as e:
            print(f"❌ Ошибка при откате: {e}")
            print(f"⚠️ Восстанавливаем из бэкапа...")
            shutil.copy2(backup_path, self.db_path)
            return False
    
    def list_backups(self) -> list:
        """Список всех бэкапов миграций."""
        backups = []
        for meta_file in self.backup_dir.glob("*_metadata.json"):
            with open(meta_file, "r") as f:
                backups.append(json.load(f))
        return sorted(backups, key=lambda x: x['timestamp'])


# Использование
migrator = SafeMigrator("alembic.ini", "myapp.db")
migrator.upgrade("head")
3.2. Автоматический откат при ошибке
python
from contextlib import contextmanager
from functools import wraps


class MigrationTransaction:
    """Транзакционный менеджер для миграций."""
    
    def __init__(self, db_path: str, alembic_cfg: str):
        self.db_path = db_path
        self.alembic_cfg = alembic_cfg
        self.backup_path = None
    
    @contextmanager
    def safe_migration(self, migration_name: str):
        """Контекстный менеджер для безопасной миграции."""
        
        # Создаём бэкап
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        self.backup_path = Path(f"backups/{timestamp}_{migration_name}.db")
        self.backup_path.parent.mkdir(exist_ok=True)
        
        conn = sqlite3.connect(self.db_path)
        backup_conn = sqlite3.connect(self.backup_path)
        conn.backup(backup_conn)
        conn.close()
        backup_conn.close()
        
        print(f"📦 Бэкап создан: {self.backup_path}")
        
        try:
            yield
            print(f"✅ Миграция {migration_name} успешно завершена")
            
        except Exception as e:
            print(f"❌ Ошибка миграции: {e}")
            print(f"🔄 Выполняется откат...")
            
            # Восстанавливаем из бэкапа
            shutil.copy2(self.backup_path, self.db_path)
            print(f"✅ Откат выполнен")
            raise
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            # Очищаем бэкап при ошибке (можно сохранить для анализа)
            pass
        else:
            # При успехе можно удалить бэкап или оставить на всякий случай
            pass


# Декоратор для миграций
def safe_migration(migration_name: str, db_path: str = "myapp.db"):
    """Декоратор для безопасного выполнения миграций."""
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            with MigrationTransaction(db_path, None).safe_migration(migration_name):
                return func(*args, **kwargs)
        return wrapper
    return decorator


# Использование
@safe_migration("add_users_table")
def run_migration():
    # Код миграции
    conn = sqlite3.connect("myapp.db")
    conn.execute("""
        CREATE TABLE users (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL
        )
    """)
    conn.commit()
    conn.close()

run_migration()
4. Стратегии отката изменений
4.1. Многоуровневый откат
python
class RollbackStrategy:
    """Стратегии отката изменений."""
    
    def __init__(self):
        self.rollback_points = []
    
    def save_rollback_point(self, name: str, data: Dict):
        """Сохраняет точку для отката."""
        self.rollback_points.append({
            "name": name,
            "data": data,
            "timestamp": datetime.now()
        })
    
    def rollback_to_last(self):
        """Откат к последней точке."""
        if self.rollback_points:
            point = self.rollback_points.pop()
            return point["data"]
        return None
    
    def rollback_to(self, name: str):
        """Откат к конкретной точке."""
        for i, point in enumerate(self.rollback_points):
            if point["name"] == name:
                self.rollback_points = self.rollback_points[:i]
                return point["data"]
        return None


class FeatureFlagRollback:
    """Управление фичами с возможностью отката."""
    
    def __init__(self, flags_file: str = "feature_flags.json"):
        self.flags_file = Path(flags_file)
        self.flags = self._load_flags()
        self.rollback = RollbackStrategy()
    
    def _load_flags(self) -> Dict:
        if self.flags_file.exists():
            with open(self.flags_file, "r") as f:
                return json.load(f)
        return {}
    
    def _save_flags(self):
        with open(self.flags_file, "w") as f:
            json.dump(self.flags, f, indent=2)
    
    def enable_feature(self, feature_name: str):
        """Включает фичу с сохранением точки отката."""
        self.rollback.save_rollback_point(f"pre_enable_{feature_name}", self.flags.copy())
        self.flags[feature_name] = True
        self._save_flags()
        print(f"✅ Фича {feature_name} включена")
    
    def disable_feature(self, feature_name: str):
        """Выключает фичу с сохранением точки отката."""
        self.rollback.save_rollback_point(f"pre_disable_{feature_name}", self.flags.copy())
        self.flags[feature_name] = False
        self._save_flags()
        print(f"✅ Фича {feature_name} выключена")
    
    def rollback_feature(self, feature_name: str):
        """Откатывает изменения фичи."""
        previous = self.rollback.rollback_to(f"pre_enable_{feature_name}")
        if previous:
            self.flags = previous
            self._save_flags()
            print(f"🔄 Фича {feature_name} откачена")
            return True
        
        previous = self.rollback.rollback_to(f"pre_disable_{feature_name}")
        if previous:
            self.flags = previous
            self._save_flags()
            print(f"🔄 Фича {feature_name} откачена")
            return True
        
        print(f"❌ Точка отката для {feature_name} не найдена")
        return False


class CanaryRollback:
    """Постепенное развёртывание с возможностью отката."""
    
    def __init__(self, config_file: str = "canary_config.json"):
        self.config_file = Path(config_file)
        self.config = self._load_config()
    
    def _load_config(self) -> Dict:
        if self.config_file.exists():
            with open(self.config_file, "r") as f:
                return json.load(f)
        return {"percentage": 0, "version": "stable"}
    
    def _save_config(self):
        with open(self.config_file, "w") as f:
            json.dump(self.config, f, indent=2)
    
    def rollout(self, percentage: int, version: str):
        """Постепенное развёртывание версии."""
        self.config["percentage"] = percentage
        self.config["version"] = version
        self._save_config()
        print(f"📊 Развёрнуто {percentage}% пользователей на версии {version}")
    
    def rollback(self):
        """Полный откат к стабильной версии."""
        self.config["percentage"] = 0
        self.config["version"] = "stable"
        self._save_config()
        print(f"🔄 Полный откат к стабильной версии")
    
    def get_version_for_user(self, user_id: str) -> str:
        """Определяет, какую версию получает пользователь."""
        import hashlib
        hash_val = int(hashlib.md5(user_id.encode()).hexdigest()[:8], 16)
        if hash_val % 100 < self.config["percentage"]:
            return self.config["version"]
        return "stable"
5. Комплексное восстановление
5.1. Disaster Recovery Plan
python
# disaster_recovery.py
import subprocess
import sys
from pathlib import Path
from datetime import datetime
from typing import Dict, List, Optional


class DisasterRecovery:
    """План восстановления после катастрофы."""
    
    def __init__(self, config_file: str = "dr_config.json"):
        self.config = self._load_config(config_file)
        self.steps_log = []
    
    def _load_config(self, config_file: str) -> Dict:
        """Загружает конфигурацию DRP."""
        default_config = {
            "backup_location": "/backups",
            "db_recovery": {
                "strategy": "point_in_time",
                "max_acceptable_data_loss_minutes": 15
            },
            "app_recovery": {
                "restore_point": "latest",
                "verification": True
            },
            "dependencies": [
                "postgresql",
                "redis",
                "nginx"
            ],
            "notification": {
                "telegram_bot": "YOUR_BOT_TOKEN",
                "chat_id": "YOUR_CHAT_ID"
            }
        }
        
        if Path(config_file).exists():
            with open(config_file, "r") as f:
                config = json.load(f)
                for key in default_config:
                    if key not in config:
                        config[key] = default_config[key]
                return config
        else:
            with open(config_file, "w") as f:
                json.dump(default_config, f, indent=2)
            return default_config
    
    def run_recovery(self, recovery_type: str = "full") -> bool:
        """
        Запускает процесс восстановления.
        
        Args:
            recovery_type: full (полное), db_only (только БД), app_only (только приложение)
        """
        print("=" * 60)
        print("🚨 ЗАПУСК ПЛАНА ВОССТАНОВЛЕНИЯ")
        print(f"   Тип: {recovery_type}")
        print(f"   Время: {datetime.now().isoformat()}")
        print("=" * 60)
        
        self._log_step("Начало восстановления")
        
        # Шаг 1: Проверка окружения
        if not self._check_environment():
            self._log_step("ОШИБКА: Окружение не готово", is_error=True)
            return False
        
        # Шаг 2: Остановка приложения
        self._stop_application()
        
        # Шаг 3: Восстановление БД
        if recovery_type in ["full", "db_only"]:
            if not self._recover_database():
                self._log_step("ОШИБКА: Не удалось восстановить БД", is_error=True)
                return False
        
        # Шаг 4: Восстановление приложения
        if recovery_type in ["full", "app_only"]:
            if not self._recover_application():
                self._log_step("ОШИБКА: Не удалось восстановить приложение", is_error=True)
                return False
        
        # Шаг 5: Восстановление зависимостей
        if not self._recover_dependencies():
            self._log_step("ПРЕДУПРЕЖДЕНИЕ: Проблемы с зависимостями", is_warning=True)
        
        # Шаг 6: Запуск приложения
        if not self._start_application():
            self._log_step("ОШИБКА: Не удалось запустить приложение", is_error=True)
            return False
        
        # Шаг 7: Верификация
        if not self._verify_application():
            self._log_step("ОШИБКА: Приложение работает некорректно", is_error=True)
            return False
        
        self._log_step("Восстановление успешно завершено")
        
        # Уведомление
        self._send_notification("✅ Восстановление завершено успешно")
        
        return True
    
    def _check_environment(self) -> bool:
        """Проверяет готовность окружения к восстановлению."""
        print("\n🔍 Проверка окружения...")
        
        checks = [
            ("Доступ к бэкапам", self._check_backups()),
            ("Доступ к сети", self._check_network()),
            ("Достаточно места на диске", self._check_disk_space()),
            ("Установлены зависимости", self._check_dependencies()),
        ]
        
        all_ok = True
        for name, result in checks:
            status = "✅" if result else "❌"
            print(f"   {status} {name}")
            if not result:
                all_ok = False
        
        return all_ok
    
    def _check_backups(self) -> bool:
        """Проверяет доступность бэкапов."""
        backup_path = Path(self.config["backup_location"])
        return backup_path.exists() and any(backup_path.iterdir())
    
    def _check_network(self) -> bool:
        """Проверяет сетевое подключение."""
        try:
            subprocess.run(["ping", "-c", "1", "8.8.8.8"], capture_output=True)
            return True
        except:
            return False
    
    def _check_disk_space(self) -> bool:
        """Проверяет свободное место на диске."""
        import shutil
        free = shutil.disk_usage("/").free
        return free > 1024 * 1024 * 100  # 100 MB
    
    def _check_dependencies(self) -> bool:
        """Проверяет установленные зависимости."""
        all_ok = True
        for dep in self.config["dependencies"]:
            try:
                if dep == "postgresql":
                    subprocess.run(["pg_isready"], capture_output=True, check=True)
                elif dep == "redis":
                    subprocess.run(["redis-cli", "ping"], capture_output=True, check=True)
                else:
                    subprocess.run([dep, "--version"], capture_output=True, check=True)
            except:
                all_ok = False
        return all_ok
    
    def _stop_application(self):
        """Останавливает приложение."""
        print("\n🛑 Остановка приложения...")
        # Здесь логика остановки приложения
        self._log_step("Приложение остановлено")
    
    def _recover_database(self) -> bool:
        """Восстанавливает базу данных."""
        print("\n💾 Восстановление базы данных...")
        
        # Ищем последний бэкап
        backup_dir = Path(self.config["backup_location"])
        backups = sorted(backup_dir.glob("*.db"), key=lambda x: x.stat().st_mtime, reverse=True)
        
        if not backups:
            print("   ❌ Бэкапы не найдены")
            return False
        
        latest_backup = backups[0]
        print(f"   📦 Используется бэкап: {latest_backup.name}")
        
        # Восстановление
        try:
            import shutil
            shutil.copy2(latest_backup, "myapp.db")
            print("   ✅ База данных восстановлена")
            self._log_step(f"БД восстановлена из {latest_backup.name}")
            return True
        except Exception as e:
            print(f"   ❌ Ошибка: {e}")
            return False
    
    def _recover_application(self) -> bool:
        """Восстанавливает приложение."""
        print("\n📦 Восстановление приложения...")
        
        # Здесь логика восстановления приложения
        # Например, из git или из бэкапа файлов
        
        self._log_step("Приложение восстановлено")
        return True
    
    def _recover_dependencies(self) -> bool:
        """Восстанавливает зависимости."""
        print("\n📚 Восстановление зависимостей...")
        
        try:
            subprocess.run(["pip", "install", "-r", "requirements.txt"], check=True)
            print("   ✅ Зависимости установлены")
            return True
        except:
            print("   ❌ Ошибка установки зависимостей")
            return False
    
    def _start_application(self) -> bool:
        """Запускает приложение."""
        print("\n🚀 Запуск приложения...")
        # Здесь логика запуска приложения
        self._log_step("Приложение запущено")
        return True
    
    def _verify_application(self) -> bool:
        """Проверяет работоспособность приложения."""
        print("\n🔍 Проверка работоспособности...")
        # Здесь логика проверки
        return True
    
    def _log_step(self, message: str, is_error: bool = False, is_warning: bool = False):
        """Логирует шаг восстановления."""
        timestamp = datetime.now().isoformat()
        level = "ERROR" if is_error else ("WARNING" if is_warning else "INFO")
        log_entry = f"[{timestamp}] {level}: {message}"
        self.steps_log.append(log_entry)
        print(log_entry)
    
    def _send_notification(self, message: str):
        """Отправляет уведомление."""
        try:
            import requests
            bot_token = self.config["notification"]["telegram_bot"]
            chat_id = self.config["notification"]["chat_id"]
            
            url = f"https://api.telegram.org/bot{bot_token}/sendMessage"
            requests.post(url, json={"chat_id": chat_id, "text": message})
        except:
            pass
    
    def get_recovery_report(self) -> str:
        """Возвращает отчёт о восстановлении."""
        return "\n".join(self.steps_log)


# Сценарии восстановления
def scenario_full_recovery():
    """Сценарий полного восстановления."""
    dr = DisasterRecovery()
    success = dr.run_recovery("full")
    
    if success:
        print("\n🎉 Система полностью восстановлена!")
    else:
        print("\n❌ Полное восстановление НЕ УДАЛОСЬ!")
    
    return success


def scenario_db_point_in_time(target_time: str):
    """Восстановление на момент времени."""
    # Требует наличия WAL и бэкапов
    print(f"⏰ Восстановление на момент времени: {target_time}")
    # Логика восстановления на момент времени


def scenario_app_version_rollback(version: str):
    """Откат приложения до версии."""
    print(f"📦 Откат приложения до версии: {version}")
    # Логика отката приложения


if __name__ == "__main__":
    scenario_full_recovery()
6. Автоматизация восстановления
6.1. Health Check и автоматический откат
python
# auto_healing.py
import time
import subprocess
import requests
from datetime import datetime
from typing import Dict, Optional


class HealthChecker:
    """Мониторинг здоровья приложения с автооткатом."""
    
    def __init__(self, config: Dict):
        self.config = config
        self.failures = 0
        self.max_failures = config.get("max_failures", 3)
        self.check_interval = config.get("check_interval", 60)
        self.last_successful_check = None
    
    def check_health(self) -> bool:
        """Проверяет здоровье приложения."""
        
        # Проверка HTTP эндпоинта
        if "http" in self.config:
            try:
                response = requests.get(
                    self.config["http"]["url"],
                    timeout=self.config["http"].get("timeout", 5)
                )
                if response.status_code == 200:
                    self._record_success()
                    return True
            except:
                pass
        
        # Проверка через пользовательскую команду
        if "command" in self.config:
            result = subprocess.run(
                self.config["command"],
                shell=True,
                capture_output=True
            )
            if result.returncode == 0:
                self._record_success()
                return True
        
        self._record_failure()
        return False
    
    def _record_success(self):
        """Фиксирует успешную проверку."""
        self.failures = 0
        self.last_successful_check = datetime.now()
    
    def _record_failure(self):
        """Фиксирует неудачную проверку."""
        self.failures += 1
        print(f"⚠️ Health check не пройден ({self.failures}/{self.max_failures})")
    
    def should_rollback(self) -> bool:
        """Определяет, нужен ли откат."""
        return self.failures >= self.max_failures
    
    def run_monitoring(self):
        """Запускает мониторинг в бесконечном цикле."""
        print(f"🔄 Запуск мониторинга (интервал: {self.check_interval} сек)")
        
        while True:
            if not self.check_health():
                if self.should_rollback():
                    print("🚨 Достигнут лимит ошибок! Запуск отката...")
                    self._perform_rollback()
                    # После отката сбрасываем счётчик
                    self.failures = 0
            
            time.sleep(self.check_interval)
    
    def _perform_rollback(self):
        """Выполняет откат приложения."""
        # Логика отката
        print("🔄 Выполняется автоматический откат...")
        
        # 1. Останавливаем текущую версию
        subprocess.run(self.config["stop_command"], shell=True)
        
        # 2. Восстанавливаем предыдущую версию
        subprocess.run(self.config["rollback_command"], shell=True)
        
        # 3. Запускаем предыдущую версию
        subprocess.run(self.config["start_command"], shell=True)
        
        print("✅ Откат выполнен")


# Конфигурация мониторинга
config = {
    "max_failures": 3,
    "check_interval": 30,
    "http": {
        "url": "http://localhost:8000/health",
        "timeout": 5
    },
    "stop_command": "systemctl stop myapp",
    "start_command": "systemctl start myapp",
    "rollback_command": "docker-compose down && docker-compose pull prev && docker-compose up -d"
}

# Запуск
# checker = HealthChecker(config)
# checker.run_monitoring()
7. Мониторинг и оповещение о сбоях
7.1. Система оповещения
python
# alerting_system.py
import smtplib
import requests
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from datetime import datetime
from typing import List, Dict


class AlertSystem:
    """Система оповещения о сбоях."""
    
    def __init__(self, config: Dict):
        self.config = config
    
    def send_telegram(self, message: str, level: str = "error"):
        """Отправка уведомления в Telegram."""
        
        icons = {"error": "🔴", "warning": "🟠", "info": "🔵"}
        icon = icons.get(level, "⚪")
        
        text = f"{icon} <b>{level.upper()}</b>\n\n{message}\n\n📅 {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}"
        
        url = f"https://api.telegram.org/bot{self.config['telegram']['bot_token']}/sendMessage"
        payload = {
            "chat_id": self.config['telegram']['chat_id'],
            "text": text,
            "parse_mode": "HTML"
        }
        
        try:
            response = requests.post(url, json=payload, timeout=10)
            return response.status_code == 200
        except:
            return False
    
    def send_email(self, subject: str, body: str, recipients: List[str]):
        """Отправка уведомления по email."""
        
        msg = MIMEMultipart()
        msg["From"] = self.config["email"]["from"]
        msg["To"] = ", ".join(recipients)
        msg["Subject"] = subject
        
        msg.attach(MIMEText(body, "html"))
        
        try:
            with smtplib.SMTP(self.config["email"]["host"], self.config["email"]["port"]) as server:
                server.starttls()
                server.login(self.config["email"]["user"], self.config["email"]["password"])
                server.send_message(msg)
            return True
        except:
            return False
    
    def notify_migration_failure(self, migration_name: str, error: str):
        """Уведомление об ошибке миграции."""
        
        message = f"""
❌ <b>ОШИБКА МИГРАЦИИ</b>

<b>Миграция:</b> {migration_name}
<b>Время:</b> {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
<b>Ошибка:</b>
<code>{error}</code>

<b>Рекомендация:</b>
1. Проверьте логи миграции
2. Выполните откат: <code>alembic downgrade -1</code>
3. Исправьте ошибку и повторите
"""
        
        self.send_telegram(message, "error")
        self.send_email("Migration Failed", message, self.config["email"]["alert_recipients"])
    
    def notify_rollback(self, from_version: str, to_version: str, reason: str):
        """Уведомление об откате."""
        
        message = f"""
⚠️ <b>ВЫПОЛНЕН ОТКАТ</b>

<b>Причина:</b> {reason}
<b>С версии:</b> {from_version}
<b>На версию:</b> {to_version}
<b>Время:</b> {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
"""
        
        self.send_telegram(message, "warning")
    
    def notify_recovery_complete(self, recovery_data: Dict):
        """Уведомление о завершении восстановления."""
        
        message = f"""
✅ <b>ВОССТАНОВЛЕНИЕ ЗАВЕРШЕНО</b>

<b>Тип:</b> {recovery_data.get('type', 'unknown')}
<b>Продолжительность:</b> {recovery_data.get('duration', 'N/A')}
<b>Потеря данных:</b> {recovery_data.get('data_loss_minutes', 0)} минут

<b>Действия:</b>
1. Проверьте работоспособность приложения
2. Проверьте целостность данных
3. Проанализируйте причины сбоя
"""
        
        self.send_telegram(message, "info")
8. Контрольные вопросы
Что такое RPO и RTO? Как они влияют на стратегию восстановления?

Как создать точку восстановления приложения в Python?

Почему важно создавать бэкап БД перед миграцией?

Как автоматически откатить миграцию при ошибке?

Что такое Canary-развёртывание и как оно связано с откатом?

Какие компоненты должны входить в план восстановления (DRP)?

Как проверить целостность точки восстановления БД?

Чем отличается полный откат от частичного?

Как настроить автоматический мониторинг здоровья приложения?

Какие метрики нужно отслеживать для своевременного обнаружения сбоя?

9. Практическое задание
Задание 1 (базовое)
Создайте скрипт, который создаёт точку восстановления для вашего приложения (копирует исходный код и БД).

Задание 2 (среднее)
Реализуйте безопасный декоратор для миграций, который автоматически создаёт бэкап перед применением и откатывает при ошибке.

Задание 3 (сложное)
Разработайте систему автоматического восстановления, которая при обнаружении сбоя:

Логирует ошибку

Отправляет уведомление

Выполняет откат к последней стабильной версии

Проверяет работоспособность после отката

10. Шпаргалка
python
# === ТОЧКИ ВОССТАНОВЛЕНИЯ ===
manager = RestorePointManager("/path/to/app")
manager.create_restore_point("Описание")
manager.restore("20250115_143025")

# === БЕЗОПАСНАЯ МИГРАЦИЯ ===
@safe_migration("migration_name")
def run_migration():
    # код миграции

# === КОНТЕКСТНЫЙ МЕНЕДЖЕР ===
with MigrationTransaction(db_path, alembic_cfg).safe_migration("name"):
    # код миграции

# === HEALTH CHECK ===
checker = HealthChecker(config)
checker.run_monitoring()

# === DRP ===
dr = DisasterRecovery()
dr.run_recovery("full")
Итог лекции
Вы сегодня:

✅ Изучили концепции RPO и RTO

✅ Научились создавать точки восстановления приложения и БД

✅ Реализовали безопасные миграции с автоматическим откатом

✅ Освоили стратегии Canary-развёртывания и отката

✅ Создали систему мониторинга и автоматического восстановления

Теперь ваше приложение готово к любым сбоям и может быть восстановлено в кратчайшие сроки!
