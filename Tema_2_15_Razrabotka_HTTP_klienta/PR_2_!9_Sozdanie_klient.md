# ПЗ 2.23. Создание клиента для работы с публичным API

**Тема:** Разработка HTTP-клиента, библиотека requests, GET, POST, PUT, DELETE запросы

**Цель работы:**  
Научиться создавать полноценного клиента для работы с публичным API, обрабатывать различные типы запросов, управлять аутентификацией, обрабатывать ошибки и пагинацию.

**Время выполнения:** 90 минут

**Необходимое ПО:**  
- VS Code с расширением Python
- Python 3.8+
- Установленная библиотека requests

```bash
pip install requests
Главная мысль: Хороший API-клиент — это не просто запросы, а удобный интерфейс для работы с внешним сервисом.

Содержание
Теоретическая справка

Нулевой вариант (эталонный)

25 вариантов практической работы

Критерии оценки

Шпаргалка

1. Теоретическая справка
1.1. Что такое API-клиент
API-клиент — это класс или модуль, который предоставляет удобный интерфейс для взаимодействия с внешним API.

python
# Без клиента — каждый раз писать запросы
response = requests.get('https://api.example.com/users/1')
user = response.json()

# С клиентом — простой и понятный код
client = APIClient()
user = client.get_user(1)
1.2. Структура API-клиента
Компонент	Назначение
Базовый URL	Адрес API (например, https://api.example.com/v1)
Сессия	Переиспользование соединений, куки, заголовки
Методы	get(), post(), put(), delete()
Обработка ошибок	Логирование и исключения
Пагинация	Получение всех страниц данных
Ретри (повторы)	Повтор запросов при временных ошибках
1.3. Публичные API для тестирования
API	Описание	Без ключа
JSONPlaceholder	Фейковые данные (посты, пользователи)	✅ Да
GitHub API	Информация о пользователях, репозиториях	✅ Да (ограниченно)
OpenWeatherMap	Погода	❌ Нет (нужен ключ)
Cat Facts	Случайные факты о котах	✅ Да
Random User	Случайные пользователи	✅ Да
PokéAPI	Информация о покемонах	✅ Да
Rick & Morty API	Информация о персонажах	✅ Да
2. Нулевой вариант (эталонный)
Назначение: преподаватель выполняет этот вариант на занятии, показывая создание клиента для публичного API.

Техническое задание (нулевой вариант)
Разработайте клиент для работы с JSONPlaceholder API (https://jsonplaceholder.typicode.com). Клиент должен поддерживать:

CRUD операции для пользователей, постов и комментариев

Пагинацию при получении списков

Обработку ошибок и логирование

Методы для поиска и фильтрации

Эталонная реализация
python
#!/usr/bin/env python3
"""
jsonplaceholder_client.py — Клиент для работы с JSONPlaceholder API.

Использование:
    from jsonplaceholder_client import JSONPlaceholderClient
    
    client = JSONPlaceholderClient()
    
    # Получение пользователей
    users = client.get_users()
    
    # Создание поста
    post = client.create_post(title="Заголовок", body="Содержание", user_id=1)
"""

import requests
import logging
from typing import List, Dict, Optional, Any
from urllib.parse import urlencode
from datetime import datetime

# Настройка логирования
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)


class JSONPlaceholderClient:
    """
    Клиент для работы с JSONPlaceholder API.
    
    Базовый URL: https://jsonplaceholder.typicode.com
    """
    
    BASE_URL = 'https://jsonplaceholder.typicode.com'
    
    def __init__(self, timeout: int = 30, log_requests: bool = False):
        """
        Инициализация клиента.
        
        Args:
            timeout: Таймаут запросов в секундах
            log_requests: Логировать ли запросы
        """
        self.timeout = timeout
        self.log_requests = log_requests
        self.logger = logging.getLogger(self.__class__.__name__)
        
        # Создаём сессию для переиспользования соединений
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'JSONPlaceholderClient/1.0',
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        })
    
    def _log_request(self, method: str, url: str, **kwargs) -> None:
        """Логирование запроса."""
        if self.log_requests:
            self.logger.info(f"{method} {url}")
            if 'json' in kwargs:
                self.logger.debug(f"Body: {kwargs['json']}")
            if 'params' in kwargs:
                self.logger.debug(f"Params: {kwargs['params']}")
    
    def _request(self, method: str, endpoint: str, **kwargs) -> Optional[Dict]:
        """
        Выполнение HTTP запроса.
        
        Args:
            method: HTTP метод (GET, POST, PUT, DELETE, PATCH)
            endpoint: Эндпоинт (например, '/users')
            **kwargs: Дополнительные параметры requests
        
        Returns:
            Ответ в виде словаря или None при ошибке
        """
        url = f"{self.BASE_URL}/{endpoint.lstrip('/')}"
        
        # Устанавливаем таймаут по умолчанию
        if 'timeout' not in kwargs:
            kwargs['timeout'] = self.timeout
        
        self._log_request(method, url, **kwargs)
        
        try:
            response = self.session.request(method, url, **kwargs)
            response.raise_for_status()
            
            # GET и DELETE могут возвращать пустой ответ
            if response.status_code == 204:
                return None
            
            return response.json()
            
        except requests.exceptions.Timeout:
            self.logger.error(f"Таймаут при запросе {method} {url}")
            return None
        except requests.exceptions.HTTPError as e:
            self.logger.error(f"HTTP ошибка {e.response.status_code}: {e.response.text}")
            return None
        except requests.exceptions.RequestException as e:
            self.logger.error(f"Ошибка запроса: {e}")
            return None
    
    # ============================================================
    # ПОЛЬЗОВАТЕЛИ (USERS)
    # ============================================================
    
    def get_users(self, **params) -> List[Dict]:
        """
        Получение списка всех пользователей.
        
        Args:
            **params: Параметры фильтрации (id, name, username, email)
        
        Returns:
            Список пользователей
        """
        result = self._request('GET', '/users', params=params)
        return result if isinstance(result, list) else []
    
    def get_user(self, user_id: int) -> Optional[Dict]:
        """
        Получение пользователя по ID.
        
        Args:
            user_id: ID пользователя
        
        Returns:
            Данные пользователя или None
        """
        return self._request('GET', f'/users/{user_id}')
    
    def create_user(self, name: str, username: str, email: str, **kwargs) -> Optional[Dict]:
        """
        Создание нового пользователя.
        
        Args:
            name: Полное имя
            username: Имя пользователя
            email: Email
            **kwargs: Дополнительные поля (address, phone, website, company)
        
        Returns:
            Созданный пользователь или None
        """
        data = {
            'name': name,
            'username': username,
            'email': email,
            **kwargs
        }
        return self._request('POST', '/users', json=data)
    
    def update_user(self, user_id: int, **data) -> Optional[Dict]:
        """
        Полное обновление пользователя.
        
        Args:
            user_id: ID пользователя
            **data: Данные для обновления
        
        Returns:
            Обновлённый пользователь или None
        """
        return self._request('PUT', f'/users/{user_id}', json=data)
    
    def patch_user(self, user_id: int, **data) -> Optional[Dict]:
        """
        Частичное обновление пользователя.
        
        Args:
            user_id: ID пользователя
            **data: Данные для обновления
        
        Returns:
            Обновлённый пользователь или None
        """
        return self._request('PATCH', f'/users/{user_id}', json=data)
    
    def delete_user(self, user_id: int) -> bool:
        """
        Удаление пользователя.
        
        Args:
            user_id: ID пользователя
        
        Returns:
            True если успешно, иначе False
        """
        result = self._request('DELETE', f'/users/{user_id}')
        return result is not None
    
    # ============================================================
    # ПОСТЫ (POSTS)
    # ============================================================
    
    def get_posts(self, user_id: Optional[int] = None, **params) -> List[Dict]:
        """
        Получение списка постов.
        
        Args:
            user_id: Фильтр по ID пользователя
            **params: Дополнительные параметры (_page, _limit, _sort)
        
        Returns:
            Список постов
        """
        if user_id:
            params['userId'] = user_id
        
        result = self._request('GET', '/posts', params=params)
        return result if isinstance(result, list) else []
    
    def get_post(self, post_id: int) -> Optional[Dict]:
        """
        Получение поста по ID.
        
        Args:
            post_id: ID поста
        
        Returns:
            Данные поста или None
        """
        return self._request('GET', f'/posts/{post_id}')
    
    def create_post(self, title: str, body: str, user_id: int) -> Optional[Dict]:
        """
        Создание нового поста.
        
        Args:
            title: Заголовок
            body: Содержание
            user_id: ID автора
        
        Returns:
            Созданный пост или None
        """
        data = {
            'title': title,
            'body': body,
            'userId': user_id
        }
        return self._request('POST', '/posts', json=data)
    
    def update_post(self, post_id: int, title: str, body: str, user_id: int) -> Optional[Dict]:
        """
        Полное обновление поста.
        
        Args:
            post_id: ID поста
            title: Заголовок
            body: Содержание
            user_id: ID автора
        
        Returns:
            Обновлённый пост или None
        """
        data = {
            'id': post_id,
            'title': title,
            'body': body,
            'userId': user_id
        }
        return self._request('PUT', f'/posts/{post_id}', json=data)
    
    def patch_post(self, post_id: int, **data) -> Optional[Dict]:
        """
        Частичное обновление поста.
        
        Args:
            post_id: ID поста
            **data: Данные для обновления (title, body, userId)
        
        Returns:
            Обновлённый пост или None
        """
        return self._request('PATCH', f'/posts/{post_id}', json=data)
    
    def delete_post(self, post_id: int) -> bool:
        """
        Удаление поста.
        
        Args:
            post_id: ID поста
        
        Returns:
            True если успешно, иначе False
        """
        result = self._request('DELETE', f'/posts/{post_id}')
        return result is not None
    
    def get_post_comments(self, post_id: int) -> List[Dict]:
        """
        Получение комментариев к посту.
        
        Args:
            post_id: ID поста
        
        Returns:
            Список комментариев
        """
        result = self._request('GET', f'/posts/{post_id}/comments')
        return result if isinstance(result, list) else []
    
    # ============================================================
    # КОММЕНТАРИИ (COMMENTS)
    # ============================================================
    
    def get_comments(self, post_id: Optional[int] = None, **params) -> List[Dict]:
        """
        Получение списка комментариев.
        
        Args:
            post_id: Фильтр по ID поста
            **params: Дополнительные параметры
        
        Returns:
            Список комментариев
        """
        if post_id:
            params['postId'] = post_id
        
        result = self._request('GET', '/comments', params=params)
        return result if isinstance(result, list) else []
    
    def get_comment(self, comment_id: int) -> Optional[Dict]:
        """
        Получение комментария по ID.
        
        Args:
            comment_id: ID комментария
        
        Returns:
            Данные комментария или None
        """
        return self._request('GET', f'/comments/{comment_id}')
    
    def create_comment(self, name: str, email: str, body: str, post_id: int) -> Optional[Dict]:
        """
        Создание комментария.
        
        Args:
            name: Имя автора
            email: Email автора
            body: Текст комментария
            post_id: ID поста
        
        Returns:
            Созданный комментарий или None
        """
        data = {
            'name': name,
            'email': email,
            'body': body,
            'postId': post_id
        }
        return self._request('POST', '/comments', json=data)
    
    def delete_comment(self, comment_id: int) -> bool:
        """
        Удаление комментария.
        
        Args:
            comment_id: ID комментария
        
        Returns:
            True если успешно, иначе False
        """
        result = self._request('DELETE', f'/comments/{comment_id}')
        return result is not None
    
    # ============================================================
    # ПАГИНАЦИЯ (ПОЛУЧЕНИЕ ВСЕХ СТРАНИЦ)
    # ============================================================
    
    def get_all_posts(self, user_id: Optional[int] = None, page_size: int = 10) -> List[Dict]:
        """
        Получение всех постов с автоматической пагинацией.
        
        Args:
            user_id: Фильтр по ID пользователя
            page_size: Размер страницы
        
        Returns:
            Все посты
        """
        all_posts = []
        page = 1
        
        while True:
            params = {
                '_page': page,
                '_limit': page_size
            }
            if user_id:
                params['userId'] = user_id
            
            posts = self.get_posts(**params)
            
            if not posts:
                break
            
            all_posts.extend(posts)
            page += 1
            
            # Если получено меньше, чем page_size — это последняя страница
            if len(posts) < page_size:
                break
        
        self.logger.info(f"Получено {len(all_posts)} постов")
        return all_posts
    
    # ============================================================
    # ПОИСК
    # ============================================================
    
    def search_posts(self, query: str) -> List[Dict]:
        """
        Поиск постов по заголовку и содержанию.
        
        Args:
            query: Поисковый запрос
        
        Returns:
            Список постов, соответствующих запросу
        """
        all_posts = self.get_all_posts()
        query_lower = query.lower()
        
        results = [
            post for post in all_posts
            if query_lower in post.get('title', '').lower()
            or query_lower in post.get('body', '').lower()
        ]
        
        self.logger.info(f"Найдено {len(results)} постов по запросу '{query}'")
        return results
    
    # ============================================================
    # СТАТИСТИКА
    # ============================================================
    
    def get_stats(self) -> Dict:
        """
        Получение статистики по API.
        
        Returns:
            Словарь со статистикой
        """
        users = self.get_users()
        posts = self.get_all_posts()
        comments = self.get_comments()
        
        return {
            'total_users': len(users),
            'total_posts': len(posts),
            'total_comments': len(comments),
            'avg_posts_per_user': len(posts) / max(len(users), 1),
            'avg_comments_per_post': len(comments) / max(len(posts), 1),
            'timestamp': datetime.now().isoformat()
        }


# ============================================================
# ПРИМЕР ИСПОЛЬЗОВАНИЯ
# ============================================================

def main():
    """Демонстрация работы клиента."""
    print("=" * 60)
    print("JSONPlaceholder API Клиент — Демонстрация")
    print("=" * 60)
    
    # Создаём клиента
    client = JSONPlaceholderClient(log_requests=True)
    
    # 1. Получение пользователей
    print("\n📋 1. Получение пользователей:")
    users = client.get_users()
    print(f"   Всего пользователей: {len(users)}")
    for user in users[:3]:
        print(f"   • {user['name']} ({user['email']})")
    
    # 2. Получение одного пользователя
    print("\n👤 2. Получение пользователя по ID (1):")
    user = client.get_user(1)
    if user:
        print(f"   {user['name']} — {user['company']['name']}")
    
    # 3. Создание нового поста
    print("\n📝 3. Создание поста:")
    new_post = client.create_post(
        title="Мой первый пост",
        body="Это содержание моего первого поста...",
        user_id=1
    )
    if new_post:
        print(f"   Пост создан! ID: {new_post['id']}")
        print(f"   Заголовок: {new_post['title']}")
    
    # 4. Получение постов пользователя
    print("\n📖 4. Посты пользователя #1:")
    posts = client.get_posts(user_id=1, _limit=3)
    for post in posts:
        print(f"   • {post['title'][:50]}...")
    
    # 5. Получение комментариев к посту
    if posts:
        print(f"\n💬 5. Комментарии к посту #{posts[0]['id']}:")
        comments = client.get_post_comments(posts[0]['id'])
        for comment in comments[:3]:
            print(f"   • {comment['name']}: {comment['body'][:40]}...")
    
    # 6. Обновление поста (PATCH)
    if new_post:
        print("\n✏️ 6. Обновление поста (PATCH):")
        updated = client.patch_post(new_post['id'], title="Обновлённый заголовок!")
        if updated:
            print(f"   Новый заголовок: {updated['title']}")
    
    # 7. Удаление поста
    if new_post:
        print("\n🗑️ 7. Удаление поста:")
        success = client.delete_post(new_post['id'])
        print(f"   Пост удалён: {success}")
    
    # 8. Статистика
    print("\n📊 8. Статистика API:")
    stats = client.get_stats()
    for key, value in stats.items():
        print(f"   • {key}: {value}")
    
    print("\n" + "=" * 60)
    print("Демонстрация завершена!")
    print("=" * 60)


if __name__ == "__main__":
    main()
Ожидаемый вывод
text
============================================================
JSONPlaceholder API Клиент — Демонстрация
============================================================

📋 1. Получение пользователей:
   Всего пользователей: 10
   • Leanne Graham (Sincere@april.biz)
   • Ervin Howell (Shanna@melissa.tv)
   • Clementine Bauch (Nathan@yesenia.net)

👤 2. Получение пользователя по ID (1):
   Leanne Graham — Romaguera-Crona

📝 3. Создание поста:
   Пост создан! ID: 101
   Заголовок: Мой первый пост

📖 4. Посты пользователя #1:
   • sunt aut facere repellat provident occaecati except...
   • qui est esse...
   • ea molestias quasi exercitationem repellat qui ipsa...

💬 5. Комментарии к посту #1:
   • id labore ex et quam laborum: Laudantium enim quasi est quidem magnam voluptate ipsam eos\...
   • quo vero reiciendis velit similique earum: Est natus enim nihil est dolore omnis voluptatem numquam et omnis occaecati quod ullam at\
   • odio adipisci rerum aut animi: Quia molestiae reprehenderit quasi aspernatur aut expedita occaecati aliquam eveniet laudantium omnis quibusdam delectus saepe quia accusamus maiores nam est\
   • ...

✏️ 6. Обновление поста (PATCH):
   Новый заголовок: Обновлённый заголовок!

🗑️ 7. Удаление поста:
   Пост удалён: True

📊 8. Статистика API:
   • total_users: 10
   • total_posts: 100
   • total_comments: 500
   • avg_posts_per_user: 10.0
   • avg_comments_per_post: 5.0
   • timestamp: 2024-01-15T14:30:25.123456

============================================================
Демонстрация завершена!
============================================================
3. 25 вариантов практической работы
Уровни сложности:

Варианты 1-8: базовый (GET запросы, один ресурс)

Варианты 9-17: средний (GET, POST, PUT, DELETE, пагинация)

Варианты 18-25: сложный (полный CRUD + поиск + статистика)

Варианты 1-8 (Базовый уровень)
№	API	Ресурс	Методы
1	Cat Facts	Факты о котах	GET /facts
2	Random User	Случайные пользователи	GET /api
3	PokéAPI	Покемоны	GET /pokemon/{name}
4	Rick & Morty	Персонажи	GET /character/{id}
5	Dog API	Породы собак	GET /breeds
6	JokeAPI	Шутки	GET /joke/Any
7	Numbers API	Факты о числах	GET /{number}
8	Agify API	Предсказание возраста	GET /?name={name}
Варианты 9-17 (Средний уровень)
№	API	Ресурс	Методы
9	JSONPlaceholder	Посты	GET, POST, PUT, DELETE
10	JSONPlaceholder	Комментарии	GET, POST, DELETE
11	JSONPlaceholder	Пользователи	GET, POST, PUT, DELETE
12	GitHub API	Пользователи	GET /users/{username}
13	GitHub API	Репозитории	GET /users/{username}/repos
14	OpenLibrary	Книги	GET /search.json?q=
15	DuckDuckGo	Поиск	GET /?q=
16	Dictionary API	Слова	GET /entries/en/{word}
17	Universities API	Университеты	GET /search?country=
Варианты 18-25 (Сложный уровень)
№	API	Требования
18	JSONPlaceholder	Полный CRUD + поиск + пагинация
19	GitHub API	Пользователи + репозитории + followers
20	OpenWeatherMap	Погода + прогноз + история
21	YouTube API	Поиск видео + информация о канале
22	Spotify API	Плейлисты + треки + поиск
23	Telegram API	Отправка сообщений + получение обновлений
24	Twitter API	Твиты + профиль + поиск
25	Google Maps API	Геокодирование + маршруты + места
4. Критерии оценки
Баллы	Критерий
2 (неудовлетворительно)	Клиент не работает или содержит критические ошибки
3 (удовлетворительно)	Реализованы GET запросы для 1-2 ресурсов
4 (хорошо)	Реализованы GET, POST, PUT, DELETE для всех ресурсов
5 (отлично)	Полный CRUD + пагинация + поиск + обработка ошибок + документация
5. Шпаргалка
5.1. Базовый шаблон клиента
python
import requests
from typing import Optional, Dict, List

class APIClient:
    BASE_URL = 'https://api.example.com'
    
    def __init__(self, timeout: int = 30):
        self.timeout = timeout
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'MyAPIClient/1.0',
            'Accept': 'application/json'
        })
    
    def _request(self, method: str, endpoint: str, **kwargs) -> Optional[Dict]:
        url = f"{self.BASE_URL}/{endpoint.lstrip('/')}"
        kwargs.setdefault('timeout', self.timeout)
        
        try:
            response = self.session.request(method, url, **kwargs)
            response.raise_for_status()
            return response.json() if response.status_code != 204 else None
        except Exception as e:
            print(f"Ошибка: {e}")
            return None
    
    def get(self, endpoint: str, **params) -> Optional[Dict]:
        return self._request('GET', endpoint, params=params)
    
    def post(self, endpoint: str, data: Dict) -> Optional[Dict]:
        return self._request('POST', endpoint, json=data)
    
    def put(self, endpoint: str, data: Dict) -> Optional[Dict]:
        return self._request('PUT', endpoint, json=data)
    
    def delete(self, endpoint: str) -> bool:
        result = self._request('DELETE', endpoint)
        return result is not None
Карточка студента
text
ПЗ 2.23. СОЗДАНИЕ КЛИЕНТА ДЛЯ РАБОТЫ С ПУБЛИЧНЫМ API

Вариант № ___
Уровень: □ Базовый □ Средний □ Сложный

API: _________________________________

=== РЕАЛИЗОВАННЫЕ МЕТОДЫ ===

□ GET /resource (список)
□ GET /resource/{id} (один)
□ POST /resource (создание)
□ PUT /resource/{id} (обновление)
□ DELETE /resource/{id} (удаление)
□ Пагинация
□ Поиск/фильтрация
□ Статистика

=== ОСОБЕННОСТИ ===

□ Аутентификация (тип: __________)
□ Rate limiting
□ Retry механизм
□ Логирование

=== ОТЧЁТ ===

Файл клиента: _____________
Примеры использования: _____________
Скриншоты работы: _____________

Дата выполнения: _____________
