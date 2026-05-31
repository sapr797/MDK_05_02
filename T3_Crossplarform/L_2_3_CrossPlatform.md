Лекция 2.3. Кроссплатформенность. Изолированные окружения. Разработка сценариев на Python и PowerShell
Цель лекции:
Научиться писать код, который работает одинаково на Windows, Linux и macOS. Создавать изолированные окружения, чтобы проекты не конфликтовали друг с другом. Писать скрипты автоматизации на Python и PowerShell.

Главная мысль: "На моём компьютере работает" — это не решение. Программа должна работать на любом компьютере.

Часть 1. Кроссплатформенность: почему это важно и как её добиться
1.1. Проблема, с которой сталкиваются все новички
Вы написали программу на Windows. Всё работает. Приносите на ноутбук с macOS или на сервер с Linux — программа падает. Почему?

Типичные причины:

Пути к файлам: C:\folder\file.txt vs /home/user/folder/file.txt

Разные командные интерпретаторы: cmd vs bash

Регистр букв в именах файлов (Linux чувствителен к регистру, Windows — нет)

Конец строки: \r\n (Windows) vs \n (Linux/macOS)

1.2. Главные принципы кроссплатформенного кода (цифрами)
3 правила, которые спасут 90% проблем:

№	Правило	Пример
1	Использовать относительные пути	./data/config.json вместо C:\myproject\data\config.json
2	Не полагаться на регистр имён файлов	open('file.txt') всегда пишите точно, как назвали файл
3	Использовать встроенные средства Python для работы с путями	os.path.join() или pathlib (см. ниже)
1.3. Правильный способ работать с путями в Python
Плохо (некроссплатформенно):

python
file = open("C:\\project\\data\\file.txt")  # Только Windows
Хорошо (работает везде):

python
import os
from pathlib import Path

# Способ 1: os.path
path = os.path.join('data', 'file.txt')

# Способ 2: pathlib (современный, предпочтительный)
path = Path('data') / 'file.txt'

# Открыть файл
with open(path, 'r') as f:
    content = f.read()
Практическое задание прямо сейчас:
Откройте терминал и выполните этот код на вашей ОС. Потом попросите соседа с другой ОС — он получит тот же результат.

Часть 2. Изолированные окружения: venv (СПАСЕНИЕ для Python-разработчика)
2.1. Проблема, которую решают изолированные окружения
Представьте:

Проект А требует requests версии 2.28

Проект Б требует requests версии 2.20
Если установить обе библиотеки глобально — они перезапишут друг друга.

Решение: каждому проекту — своё изолированное окружение с собственными библиотеками.

2.2. venv — встроенный инструмент Python (не требует установки)
Пошаговая инструкция (выполните прямо сейчас):

bash
# 1. Создаём папку проекта
mkdir my_isolated_project
cd my_isolated_project

# 2. Создаём виртуальное окружение
python -m venv venv

# 3. АКТИВИРУЕМ окружение (важно!)
# На Windows:
venv\Scripts\activate

# На macOS / Linux:
source venv/bin/activate

# 4. Проверяем, что активировалось
# В начале строки терминала появится (venv)
which python   # Linux/macOS
where python   # Windows
Что произошло: теперь pip install будет ставить библиотеки только в папку venv/, а не глобально.

2.3. Управление зависимостями (requirements.txt)
Когда вы настроили окружение и установили нужные библиотеки, нужно сохранить их список:

bash
# Активируем окружение (если ещё не активировали)
source venv/bin/activate   # или venv\Scripts\activate

# Устанавливаем библиотеки
pip install requests pandas flask

# Сохраняем список всех зависимостей
pip freeze > requirements.txt
Файл requirements.txt выглядит так:

text
requests==2.31.0
pandas==2.1.0
flask==3.0.0
Другой разработчик (или вы сами на другом компьютере) может воссоздать окружение одной командой:

bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
2.4. Жизненный цикл изолированного окружения (цифрами)
Этап	Действие	Команда
1	Создать	python -m venv venv
2	Активировать	source venv/bin/activate
3	Установить библиотеки	pip install <package>
4	Сохранить зависимости	pip freeze > requirements.txt
5	Выйти из окружения	deactivate
Важное правило: папку venv/ НЕ добавляют в Git (она в .gitignore). В репозиторий кладут только requirements.txt.

Часть 3. Разработка сценариев на Python
3.1. Что такое "сценарий" (скрипт) в контексте ИС
Сценарий — это программа, которая автоматизирует рутинные задачи:

Обработка файлов (переименовать, переместить, преобразовать)

Работа с базами данных (бэкап, очистка)

Взаимодействие с API (загрузить данные, отправить отчёт)

Тестирование и деплой

Отличие от "обычной программы": сценарий — это инструмент для разработчика или администратора, а не для конечного пользователя.

3.2. Пример 1: скрипт для очистки временных файлов
Создайте файл clean_temp.py:

python
#!/usr/bin/env python3
"""
Скрипт для удаления временных файлов из папки temp.
Работает на Windows, Linux, macOS.
"""

import os
import sys
from pathlib import Path

def clean_temp_folder(folder_path, extensions):
    """
    Удаляет файлы с указанными расширениями из папки.
    
    Args:
        folder_path: путь к папке
        extensions: список расширений (например, ['.tmp', '.log'])
    """
    folder = Path(folder_path)
    
    if not folder.exists():
        print(f"Ошибка: папка {folder_path} не существует")
        return 0
    
    deleted_count = 0
    
    for ext in extensions:
        for file in folder.glob(f"*{ext}"):
            try:
                file.unlink()
                print(f"Удалён: {file.name}")
                deleted_count += 1
            except Exception as e:
                print(f"Ошибка при удалении {file.name}: {e}")
    
    return deleted_count

if __name__ == "__main__":
    # Параметры скрипта
    target_folder = sys.argv[1] if len(sys.argv) > 1 else "./temp"
    extensions_to_delete = ['.tmp', '.log', '.cache', '.pyc']
    
    print(f"Очистка папки: {target_folder}")
    count = clean_temp_folder(target_folder, extensions_to_delete)
    print(f"Удалено файлов: {count}")
Как запустить:

bash
python clean_temp.py ./temp
3.3. Пример 2: скрипт для бэкапа с кроссплатформенными путями
python
#!/usr/bin/env python3
"""
Скрипт для создания бэкапов.
Использует pathlib для кроссплатформенности.
"""

import shutil
import datetime
from pathlib import Path

def create_backup(source_dir, backup_dir):
    """
    Создаёт копию source_dir в backup_dir с датой в имени.
    """
    source = Path(source_dir)
    backup = Path(backup_dir)
    
    # Создаём имя папки бэкапа с текущей датой
    date_str = datetime.datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
    backup_path = backup / f"backup_{date_str}"
    
    # Проверяем исходную папку
    if not source.exists():
        raise FileNotFoundError(f"Исходная папка не найдена: {source}")
    
    # Создаём папку для бэкапа
    backup_path.mkdir(parents=True, exist_ok=True)
    
    # Копируем
    for item in source.iterdir():
        if item.is_file():
            shutil.copy2(item, backup_path / item.name)
        elif item.is_dir():
            shutil.copytree(item, backup_path / item.name)
    
    print(f"Бэкап создан: {backup_path}")
    return backup_path

if __name__ == "__main__":
    create_backup("./data", "./backups")
Часть 4. Разработка сценариев на PowerShell
4.1. Когда нужен PowerShell, если есть Python
PowerShell — это не "ещё один язык", а встроенный инструмент Windows для администрирования. Он нужен, когда:

Вы работаете с Windows-специфичными вещами (реестр, службы, Active Directory)

Вы пишете скрипты для CI/CD на Windows-серверах

Ваши коллеги-администраторы не знают Python

Важно: PowerShell теперь кроссплатформенный (PowerShell 7+ работает на Linux и macOS).

4.2. Основные команды PowerShell (для тех, кто знает CMD или Bash)
Задача	CMD / Bash	PowerShell
Посмотреть файлы	dir / ls	Get-ChildItem (или ls, dir)
Переместить файл	move / mv	Move-Item
Удалить файл	del / rm	Remove-Item
Создать папку	mkdir	New-Item -ItemType Directory
Вывести текст	echo	Write-Host
Переменная	%VAR% / $VAR	$var
4.3. Пример: скрипт PowerShell для очистки временных файлов (кроссплатформенно)
powershell
<#
.SYNOPSIS
Очищает временные файлы в указанной папке.

.DESCRIPTION
Удаляет файлы с расширениями .tmp, .log, .cache.
Работает в PowerShell 7+ на Windows, Linux, macOS.
#>

param(
    [Parameter(Mandatory=$false)]
    [string]$TargetFolder = "./temp"
)

# Получаем абсолютный путь (кроссплатформенно)
$fullPath = Resolve-Path $TargetFolder -ErrorAction SilentlyContinue

if (-not $fullPath) {
    Write-Host "Ошибка: папка $TargetFolder не существует" -ForegroundColor Red
    exit 1
}

$extensions = @(".tmp", ".log", ".cache")
$deletedCount = 0

foreach ($ext in $extensions) {
    $files = Get-ChildItem -Path $fullPath -Filter "*$ext" -File
    
    foreach ($file in $files) {
        Remove-Item -Path $file.FullName -Force
        Write-Host "Удалён: $($file.Name)" -ForegroundColor Green
        $deletedCount++
    }
}

Write-Host "Удалено файлов: $deletedCount" -ForegroundColor Cyan
Как запустить:

powershell
# Сохранить как clean_temp.ps1
.\clean_temp.ps1 -TargetFolder "./temp"
4.4. Выбор между Python и PowerShell (таблица решений)
Задача	Лучше Python	Лучше PowerShell
Сложная логика (условия, циклы, классы)	✅	—
Работа с REST API, JSON, CSV	✅	—
Машинное обучение, анализ данных	✅	—
Управление реестром Windows	—	✅
Управление IIS, Active Directory	—	✅
Простая обработка файлов (скопировать, удалить)	—	✅
Скрипт должен работать на любом компьютере без установки	✅ (Python должен быть установлен)	✅ (PowerShell 7+ должен быть установлен)
Часть 5. Сквозной практикум: кроссплатформенный проект
Задание: написать скрипт, который собирает информацию о системе
Требования:

Работает на Windows, Linux, macOS

Использует изолированное окружение (venv)

Сохраняет результат в JSON

Имеет обработку ошибок

Решение (файл system_info.py):

python
#!/usr/bin/env python3
"""
Кроссплатформенный скрипт для сбора информации о системе.
"""

import platform
import os
import json
import datetime
from pathlib import Path
import subprocess

def get_os_info():
    """Возвращает информацию об ОС."""
    return {
        "system": platform.system(),
        "release": platform.release(),
        "version": platform.version(),
        "machine": platform.machine(),
        "processor": platform.processor()
    }

def get_cpu_info():
    """Возвращает информацию о процессоре (кроссплатформенно)."""
    cpu_count = os.cpu_count()
    
    # Для Linux/macOS
    if platform.system() != "Windows":
        try:
            with open("/proc/cpuinfo") as f:
                model = [line for line in f if "model name" in line][0].strip().split(":")[1].strip()
            return {"cores": cpu_count, "model": model}
        except:
            pass
    
    return {"cores": cpu_count, "model": "Не определён"}

def get_memory_info():
    """Возвращает информацию о памяти (кроссплатформенно)."""
    mem_info = {}
    
    if platform.system() == "Windows":
        # Windows через systeminfo
        try:
            result = subprocess.run(["systeminfo"], capture_output=True, text=True)
            for line in result.stdout.split("\n"):
                if "Total Physical Memory" in line:
                    mem_info["total_mb"] = line.split(":")[1].strip()
                    break
        except:
            pass
    else:
        # Linux/macOS
        try:
            with open("/proc/meminfo") as f:
                for line in f:
                    if "MemTotal" in line:
                        total_kb = int(line.split(":")[1].strip().split()[0])
                        mem_info["total_mb"] = f"{total_kb // 1024} MB"
                        break
        except:
            pass
    
    return mem_info or {"total_mb": "Не определён"}

def save_report(data, filename="system_report.json"):
    """Сохраняет отчёт в JSON."""
    report_dir = Path("./reports")
    report_dir.mkdir(exist_ok=True)
    
    timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
    filepath = report_dir / f"{filename}_{timestamp}.json"
    
    with open(filepath, 'w', encoding='utf-8') as f:
        json.dump(data, f, indent=2, ensure_ascii=False)
    
    print(f"Отчёт сохранён: {filepath}")
    return filepath

if __name__ == "__main__":
    print("Сбор информации о системе...")
    
    report = {
        "timestamp": datetime.datetime.now().isoformat(),
        "os": get_os_info(),
        "cpu": get_cpu_info(),
        "memory": get_memory_info()
    }
    
    print(json.dumps(report, indent=2, ensure_ascii=False))
    save_report(report)
Запуск проекта (полный цикл):
bash
# 1. Создаём папку и окружение
mkdir crossplatform_project
cd crossplatform_project
python -m venv venv

# 2. Активируем
source venv/bin/activate   # или venv\Scripts\activate

# 3. Сохраняем зависимости (скрипт использует только стандартную библиотеку)
pip freeze > requirements.txt

# 4. Запускаем скрипт
python system_info.py

# 5. Проверяем результат
cat reports/system_report_*.json
Практическая работа (ПЗ 2.3). Кроссплатформенные сценарии
ПР 2.3 состоит из трёх самостоятельных заданий. Выполнить все.

Задание 1. Настройка изолированного окружения (15 минут)
Создайте папку env_practice.

Создайте виртуальное окружение venv.

Активируйте его.

Установите библиотеки: requests, pandas, python-dotenv.

Сохраните зависимости в requirements.txt.

Создайте файл .gitignore и добавьте в него venv/.

Деактивируйте окружение.

Что сдать: скриншот вывода pip freeze и содержимого .gitignore.

Задание 2. Кроссплатформенный скрипт-валидатор (30 минут)
Напишите скрипт file_validator.py, который:

Принимает через командную строку путь к папке

Проходит по всем файлам .txt в этой папке

Проверяет, что файл не пустой и содержит слово "VALID"

Создаёт отчёт validation_report.json с результатами

Требования:

Использовать pathlib (кроссплатформенность)

Обрабатывать ошибки (нет папки, нет прав)

Сохранять JSON с кодировкой UTF-8

Пример запуска:

bash
python file_validator.py ./documents
Задание 3. PowerShell или Python (на выбор) (30 минут)
Вариант А (PowerShell):
Создайте скрипт clean_old_backups.ps1, который удаляет папки бэкапов старше 30 дней в указанной директории.

Вариант Б (Python):
Допишите system_info.py, добавив функцию get_network_info(), которая определяет IP-адрес компьютера (кроссплатформенно через socket или subprocess).

Отчёт по ПР 2.3
В отчёт включить:

Скриншот терминала с активированным venv и установленными пакетами.

Код скрипта file_validator.py и пример validation_report.json.

Код выбранного скрипта из задания 3.

Ответы на контрольные вопросы:

Почему os.path.join() лучше, чем конкатенация строк с / или \?

Что будет, если закоммитить папку venv в Git? (экспериментально проверить)

В каком случае вы выберете PowerShell, а в каком — Python для скрипта автоматизации?

Критерии оценки:

«Зачтено» — задания 1 и 2 выполнены полностью.

«Отлично» — выполнено задание 3, код соответствует PEP8, есть обработка ошибок.

Итог лекции 2.3
Вы сегодня:

Поняли, почему код может не работать на другой ОС и как это исправить.

Научились создавать изолированные окружения через venv — теперь ваши проекты не конфликтуют.

Освоили requirements.txt для воспроизводимости окружений.

Написали кроссплатформенные скрипты на Python.

Познакомились с PowerShell как альтернативой для Windows-администрирования.

Теперь вы умеете создавать проекты, которые готовы к переносу на любую платформу. 
