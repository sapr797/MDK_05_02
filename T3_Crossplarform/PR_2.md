ПР 2.2. Настройка изолированного окружения Python (venv) и создание requirements. Управление зависимостями: установка, обновление, заморозка
Цель работы:
Научиться создавать изолированные Python-окружения, управлять зависимостями проекта и фиксировать их с помощью requirements.txt. Это основа для любого серьёзного Python-проекта.

Что вы получите в итоге:

Понимание, как не "поломать" один проект другим.

Умение переносить проект на другой компьютер одной командой.

Готовый шаблон (папка + окружение + requirements), который можно использовать всегда.

Время выполнения: 60 минут.

Необходимое ПО:

Python 3.8 или выше (проверьте: python --version в терминале)

VS Code с расширением Python (установили в ПР 2.1)

Терминал (встроенный в VS Code)

Часть 1. Создание изолированного окружения (15 минут)
Шаг 1. Создаём папку проекта
Откройте терминал в VS Code (Ctrl+`) и выполните:

bash
# Создаём и переходим в папку
mkdir my_first_env
cd my_first_env

# Проверяем, что мы в правильном месте
pwd        # macOS/Linux
cd         # Windows
Шаг 2. Создаём виртуальное окружение
bash
python -m venv venv
Что произошло:
Python создал папку venv/, внутри которой лежит отдельная копия интерпретатора и отдельная папка для библиотек.

Проверка: выполните ls (macOS/Linux) или dir (Windows). Вы увидите папку venv.

Шаг 3. Активируем окружение
На Windows:

bash
venv\Scripts\activate
На macOS / Linux:

bash
source venv/bin/activate
Как проверить, что активировалось:

В начале строки терминала появится (venv)

Команда which python (macOS/Linux) или where python (Windows) покажет путь внутри venv/

Важно: активация действует только в этом окне терминала. При закрытии терминала окружение деактивируется.

Шаг 4. Проверяем чистоту окружения
bash
# Список установленных библиотек (должен быть пустым или почти пустым)
pip list
Вы должны увидеть только pip и setuptools. Никаких django, flask, requests — окружение чистое.

Фиксация результата: сделайте скриншот вывода pip list с (venv) в строке терминала.

Часть 2. Установка зависимостей (15 минут)
Шаг 1. Устанавливаем несколько библиотек
bash
# Установка трёх популярных библиотек
pip install requests
pip install pandas
pip install flask
Альтернатива (одной командой):

bash
pip install requests pandas flask
Шаг 2. Проверяем установку
bash
# Смотрим, что теперь есть в окружении
pip list
Теперь вы увидите requests, pandas, flask и их зависимости (например, numpy, click, Jinja2).

Шаг 3. Пишем простую программу, использующую библиотеки
Создайте в VS Code файл app.py (в той же папке my_first_env):

python
"""
Простая программа для проверки работы зависимостей.
Использует requests для HTTP-запроса и pandas для данных.
"""

import requests
import pandas as pd
from flask import Flask

print("=== Проверка установленных библиотек ===")
print(f"Версия requests: {requests.__version__}")
print(f"Версия pandas: {pd.__version__}")
print(f"Flask импортирован успешно")

# Простой тест requests
response = requests.get("https://httpbin.org/get")
print(f"\nТест requests: статус {response.status_code}")

# Простой тест pandas
data = {'Name': ['Python', 'venv', 'requirements'], 'Status': ['OK', 'OK', 'OK']}
df = pd.DataFrame(data)
print(f"\nТест pandas:\n{df}")

print("\nОкружение работает корректно!")
Запустите программу:

bash
python app.py
Результат: программа должна выполниться без ошибок.

Фиксация результата: скриншот работающей программы.

Часть 3. Заморозка зависимостей (15 минут)
Шаг 1. Создаём requirements.txt
bash
pip freeze > requirements.txt
Шаг 2. Смотрим содержимое файла
bash
# macOS/Linux
cat requirements.txt

# Windows
type requirements.txt
Вы увидите что-то похожее:

text
blinker==1.6.2
click==8.1.7
Flask==3.0.0
itsdangerous==2.1.2
Jinja2==3.1.2
MarkupSafe==2.1.3
numpy==1.24.3
pandas==2.1.0
python-dateutil==2.8.2
pytz==2023.3
requests==2.31.0
six==1.16.0
tzdata==2023.3
Werkzeug==3.0.0
Что это значит: каждая библиотека зафиксирована с конкретной версией (оператор ==). Это гарантирует, что при установке на другом компьютере версии будут точно такие же.

Шаг 3. Создаём .gitignore (чтобы не закоммитить окружение)
Создайте файл .gitignore:

bash
# macOS/Linux
echo "venv/" > .gitignore
echo "__pycache__/" >> .gitignore
echo "*.pyc" >> .gitignore

# Windows (в PowerShell)
"venv/" | Out-File -FilePath .gitignore -Encoding utf8
"__pycache__/" | Out-File -FilePath .gitignore -Encoding utf8 -Append
"*.pyc" | Out-File -FilePath .gitignore -Encoding utf8 -Append
Содержимое .gitignore должно быть:

text
venv/
__pycache__/
*.pyc
Почему это важно: папка venv/ может занимать сотни мегабайт, её не нужно хранить в Git. Достаточно requirements.txt.

Фиксация результата: скриншот содержимого requirements.txt и .gitignore.

Часть 4. Восстановление окружения из requirements.txt (15 минут)
Этот сценарий нужен, когда вы или другой разработчик клонирует проект с Git.

Шаг 1. Имитируем получение проекта "с нуля"
Закройте терминал и откройте новый (или выполните деактивацию):

bash
deactivate
Теперь вы снова в глобальном окружении Python.

Шаг 2. Создаём новое окружение для этого же проекта
bash
# Убедитесь, что вы в папке my_first_env
cd my_first_env

# Удаляем старое окружение (опционально, для чистоты эксперимента)
rm -rf venv        # macOS/Linux
rmdir /s venv      # Windows

# Создаём новое чистое окружение
python -m venv venv

# Активируем
source venv/bin/activate   # или venv\Scripts\activate
Шаг 3. Устанавливаем зависимости из requirements.txt
bash
pip install -r requirements.txt
Что происходит: pip читает файл requirements.txt и устанавливает все библиотеки с теми же версиями, которые были зафиксированы.

Шаг 4. Проверяем, что всё работает
bash
python app.py
Программа должна выполниться так же, как и раньше, без ошибок.

Фиксация результата: скриншот успешного запуска после установки из requirements.txt.

Часть 5. Управление зависимостями (дополнительно для "отлично")
Задание 5.1. Обновление зависимостей
bash
# Проверяем устаревшие пакеты
pip list --outdated

# Обновляем конкретный пакет (например, requests)
pip install --upgrade requests

# Обновляем ВСЕ пакеты (осторожно — может сломать совместимость)
pip install --upgrade pip
# Для массового обновления используют pip-review (требует установки)
pip install pip-review
pip-review --auto
Задание 5.2. Разные requirements для разных сред
В реальных проектах часто используют несколько файлов:

text
requirements/
├── base.txt      # общие зависимости
├── dev.txt       # для разработки (pytest, black, mypy)
└── prod.txt      # для продакшена (только необходимое)
Пример base.txt:

text
requests==2.31.0
pandas==2.1.0
Пример dev.txt:

text
-r base.txt
pytest==7.4.0
black==23.9.0
Установка для разработки:

bash
pip install -r requirements/dev.txt
Попробуйте создать такую структуру самостоятельно.

Задание 5.3. Обработка конфликтов версий
Иногда две библиотеки требуют разные версии одной зависимости. Проверить конфликты:

bash
pip check
Если конфликтов нет — увидите No broken requirements found.

Контрольные вопросы (для отчёта)
Ответьте письменно (2-3 предложения на вопрос):

Что произойдёт, если запустить pip freeze > requirements.txt без активации venv?
Подсказка: попробуйте выполнить (вне окружения) и сравните результат.

Почему в requirements.txt указываются не только явно установленные библиотеки (requests, pandas), но и их зависимости (numpy, click и др.)?

В чём разница между pip freeze и pip list? Когда что использовать?

Что означает флаг -r в команде pip install -r requirements.txt?

Ваш коллега закоммитил папку venv/ в Git. Какие проблемы это создаст?

Критерии оценки
Баллы	Критерий
3 (зачтено)	Выполнены Части 1-3: создано окружение, установлены библиотеки, создан requirements.txt
4 (хорошо)	+ выполнена Часть 4: восстановление окружения из requirements
5 (отлично)	+ выполнено любое задание из Части 5 (обновление, раздельные requirements, проверка конфликтов)
Шпаргалка (можно сохранить)
bash
# === СОЗДАНИЕ ОКРУЖЕНИЯ ===
python -m venv venv                    # создать
source venv/bin/activate               # активировать (macOS/Linux)
venv\Scripts\activate                  # активировать (Windows)
deactivate                             # выйти из окружения

# === УПРАВЛЕНИЕ ПАКЕТАМИ ===
pip install <package>                  # установить
pip install <package>==1.2.3           # установить конкретную версию
pip install --upgrade <package>        # обновить
pip uninstall <package>                # удалить
pip list                               # посмотреть все
pip list --outdated                    # устаревшие пакеты
pip show <package>                     # информация о пакете

# === ЗАВИСИМОСТИ ===
pip freeze > requirements.txt          # сохранить
pip install -r requirements.txt        # восстановить
pip check                              # проверить конфликты
Что дальше?
Теперь вы умеете:

Создавать изолированные окружения

Устанавливать и обновлять зависимости

Фиксировать и восстанавливать окружение через requirements.txt

Этот навык нужен для:

Работы в команде (все используют одни и те же версии библиотек)

Деплоя проектов на сервер

Контейнеризации (Docker) — следующий шаг

Следующая практическая работа — работа с Git и удалёнными репозиториями (GitHub). Вы научитесь пушить проекты вместе с requirements.txt (но без папки venv).
