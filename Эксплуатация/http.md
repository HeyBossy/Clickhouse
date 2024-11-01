 Когда мы хотим добавить данные в ClickHouse из внешнего источника, нам зачастую необходимо написать дополнительный код который вставит данные. У ClickHouse есть 2 основных протокола через которые мы можем добавить данные, это HTTP-текстовый протокол, то есть возможность отправить текст через интернет, и нативный протокол для обмена бинарными данными(в следующем уроке мы разберемся как с ним работать).

**HTTP (Hypertext Transfer Protocol)** - это протокол, используемый для передачи данных в Интернете в текстовом виде. Он определяет, как клиент(тот кто отправляет) и сервер(тот кто принимает) должны общаться друг с другом. В ClickHouse HTTP протокол открыт на порту ``8123`` давайте попробуем запустить Google Colab и написать несколько запросов к нему.

# Пример
Начнем с простого запроса и командной утилиты которая позволяет отправлять текстовый запрос по HTTP протоколу.
```python
!curl 'http://localhost:8123/ping'
```
Итак ответ получен значит все корректно настроено и работает. Напишем простой select запрос.
```python
SELECT%201 %20 = пробел

!curl 'http://localhost:8123?query=SELECT%201'
```

База данныз треубет указать пользователя и пароль, чтобы получить доступ к таблице, для этого нужно дополнительно прописать следующие атрибуты.

```python
!curl 'http://localhost:8123?user=default&password=12345&query=SELECT%201'
1
```

Давайте теперь напишем Python код который делает тоже самое, и создает таблицу.
```python
import requests

# Определяем параметры подключения к ClickHouse
url = 'http://localhost:8123/'

# Определяем запрос на создание таблицы
create_table_query = f"CREATE TABLE my_table (id Int64, desc String) ENGINE = Log"

# Отправляем запрос на создание таблицы
response = requests.post(url, params={'query': create_table_query, "user": "default", "password": 12345})

# Проверяем статус-код ответа
if response.status_code == 200:
    print('Таблица успешно создана')
else:
    print(f'Ошибка при создании таблицы: {response.text}')
```

Теперь вставим данные в эту таблицу
```python
import requests

# Определяем параметры подключения к ClickHouse
url = 'http://localhost:8123/'


# Определяем запрос на вставку данных
insert_query = "INSERT INTO my_table (*) VALUES (1, 'text'), (2, 'text2')"


# Отправляем запрос на создание таблицы
response = requests.post(url, params={'query': insert_query, "user": "default", "password": 12345})

# Проверяем статус-код ответа
if response.status_code == 200:
    print('Данные вставились')
else:
    print(f'Ошибка вставки: {response.text}')
    
Данные вставились
```

Теперь напишем запрос который выведет полученную таблицу
```python
import requests

# Определяем параметры подключения к ClickHouse
url = 'http://localhost:8123/'

# Определяем запрос на вставку данных
select_query = f"select * from my_table"


# Отправляем запрос на создание таблицы
response = requests.post(url, params={'query': select_query, "user": "default", "password": 12345})

# Проверяем статус-код ответа
if response.status_code == 200:
    print(response.text)
else:
    print(f'Ошибка вставки: {response.text}')
    
1	text
2	text2
```