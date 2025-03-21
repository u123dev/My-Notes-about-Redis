# My Notes about Redis
My experience in projects & some cases to help  to keep in mind Redis basic 

* [How to Run Redis](#How)
* [List of Redis commands](#List)
* [RDB (Redis Database) & AOF (Append Only File) Modes](#RDB-AOF)
* [Examples of Redis commands on Python](#Examples)
* [Some cases using Redis](#Cases)
* [Example - how to use Redis with the popular Redis client for Python redis-py](#Client)
* [Recent case - Redis as Vector Database](#Vector)

<a id="How"></a>
## How to Run / Redis :
* #### Redis in Docker
```bash
docker run -d --name redis-container -p 6379:6379 redis
```
* #### Redis CLI in docker
```bash
docker exec -it redis-container redis-cli
```
* #### To install ```redis-py``` :
```bash
pip install redis
```
* #### Connecting to a default Redis instance, running locally.
```python
import redis

connection = redis.Redis()
connection.ping()    # True
```

* #### Connecting to a redis instance, specifying a host and port with credentials.
```python
import redis

user_connection = redis.Redis(host='localhost', port=6380, username='dvora', password='redis', decode_responses=True)
user_connection.ping()   # True
```
* #### Connecting to a redis instance with username and password credential provider
```python
import redis

creds_provider = redis.UsernamePasswordCredentialProvider("username", "password")
user_connection = redis.Redis(host="localhost", port=6379, credential_provider=creds_provider)
user_connection.ping()   # True
```

<a id="List"></a>
# List of Redis commands commonly used in Python applications
To use with the ```redis-py``` module:

* ## String Commands / Redis
```python
redis.set(key, value)            → Устанавливает значение value в ключ key.
redis.get(key)                   → Получает значение из key.
redis.delete(*keys)              → Удаляет один или несколько ключей.
redis.incr(key, amount=1)        → Увеличивает числовое значение key на amount (по умолчанию +1).
redis.decr(key, amount=1)        → Уменьшает числовое значение key на amount (по умолчанию -1).
redis.exists(key)                → Проверяет, существует ли ключ (1, если да, 0, если нет).
redis.expire(key, seconds)       → Устанавливает время жизни key в секундах.
redis.ttl(key)                   → Возвращает оставшееся время жизни ключа (в секундах).
redis.keys(pattern)              → Ищет все ключи, подходящие под шаблон (например, "user:*").
redis.type(key)                  → Возвращает тип данных ключа (string, list, set и т. д.).
redis.rename(old_key, new_key)   → Переименовывает old_key в new_key (перезаписывает, если новый ключ уже есть).
redis.renamenx(old_key, new_key) → Переименовывает, только если new_key не существует.
redis.move(key, db)              → Перемещает ключ key в другую базу данных (db).
redis.mset(mapping)              → Записывает несколько ключей сразу (пример: {"k1": "v1", "k2": "v2"}).
redis.mget(keys)                 → Получает значения сразу для нескольких ключей.
```

* ## Hash Commands / Redis
Хэши в Redis — это структуры данных, хранящие пары ключ-значение внутри одного ключа.
```python
redis.hset(name, key, value)              → Устанавливает поле key со значением value в хэш name.
redis.hget(name, key)                     → Получает значение поля key из хэша name.
redis.hgetall(name)                       → Получает весь хэш name (ключи + значения).
redis.hdel(name, *keys)                   → Удаляет одно или несколько полей из хэша.
redis.hexists(name, key)                  → Проверяет, существует ли поле key в хэше (1, если да, 0, если нет).
redis.hkeys(name)                         → Получает все ключи (поля) внутри хэша.
redis.hlen(name)                          → Возвращает количество полей в хэше.
redis.hmget(name, keys)                   → Получает значения сразу для нескольких полей.
redis.hmset(name, mapping)                → Устанавливает несколько полей сразу ({"k1": "v1", "k2": "v2"}).
redis.hvals(name)                         → Получает все значения хэша.
redis.hincrby(name, key, amount=1)        → Увеличивает числовое значение поля key на amount.
redis.hincrbyfloat(name, key, amount=1.0) → Увеличивает числовое значение поля key на amount (поддерживает float).
```
Пример:
```python
redis.hset("user:1001", "name", "Alice")
redis.hget("user:1001", "name")  # Вернёт "Alice"
Хэши удобны для хранения объектов, например, данных пользователя.
```

* ## List Commands / Redis
Списки в Redis — это упорядоченные коллекции элементов (аналог очереди или стека).
```python
redis.rpush(key, *values)            → Добавляет values в конец списка key.
redis.lpush(key, *values)            → Добавляет values в начало списка key.
redis.llen(key)                      → Возвращает длину списка key.
redis.lrange(key, start, end)        → Получает элементы списка с start до end (например, lrange("mylist", 0, -1) вернёт весь список).
redis.lindex(key, index)             → Получает элемент по индексу index в списке.
redis.lset(key, index, value)        → Устанавливает значение value в index списка.
redis.lrem(key, value, num=0)        → Удаляет num вхождений value из списка (num > 0 — с начала, < 0 — с конца, 0 — все).
redis.lpop(key)                      → Удаляет и возвращает первый элемент списка.
redis.rpop(key)                      → Удаляет и возвращает последний элемент списка.
redis.rpoplpush(source, destination) → Перемещает последний элемент из source в начало destination.
```
Пример:
```python
redis.rpush("queue", "task1", "task2")
redis.lpop("queue")  # Вернёт "task1" и удалит его
Списки полезны для реализации очередей и стэков.
```

* ## Set Commands / Redis 
Множества (Set) в Redis — это неупорядоченные коллекции уникальных элементов.
```python
redis.sadd(key, *values)            → Добавляет values в множество key (уникальные значения).
redis.srem(key, *values)            → Удаляет values из множества key.
redis.spop(key)                     → Удаляет и возвращает случайный элемент множества.
redis.srandmember(key, number=None) → Возвращает number случайных элементов множества (без удаления).
redis.sismember(key, value)         → Проверяет, есть ли value в множестве (1, если да, 0, если нет).
redis.scard(key)                    → Возвращает количество элементов в множестве.
redis.smembers(key)                 → Возвращает все элементы множества.
redis.sunion(keys, *args)           → Возвращает объединение множеств keys.
redis.sinter(keys, *args)           → Возвращает пересечение множеств keys.
redis.sdiff(keys, *args)            → Возвращает разницу множеств (keys[0] - keys[1] - ...).
```
Пример:
```python
redis.sadd("users", "Alice", "Bob", "Charlie")
redis.sismember("users", "Alice")  # Вернёт 1
redis.srem("users", "Bob")
```
Множества удобны для хранения уникальных значений (например, ID пользователей).


* ## Sorted Set Commands / Redis
Отсортированные множества (Sorted Set, ZSET) — это множества с уникальными значениями, но каждый элемент имеет числовой "вес" (score), который определяет порядок сортировки.
```python
redis.zadd(name, mapping)                             → Добавляет элементы в name с их score (пример: {"Alice": 10, "Bob": 20}).
redis.zrem(name, *values)                             → Удаляет values из множества name.
redis.zrange(name, start, end, withscores=False)      → Возвращает элементы по индексу (от start до end, 0 — первый, -1 — последний).
redis.zrevrange(name, start, end, withscores=False)   → То же, но в обратном порядке.
redis.zrangebyscore(name, min, max, withscores=False) → Возвращает элементы по диапазону score (min ≤ score ≤ max).
redis.zcard(name)                                     → Количество элементов в множестве.
redis.zscore(name, value)                             → Возвращает score элемента value.
redis.zincrby(name, value, amount=1)                  → Увеличивает score элемента value на amount.
redis.zcount(name, min, max)                          → Количество элементов с score в диапазоне [min, max].
redis.zremrangebyrank(name, start, end)               → Удаляет элементы по индексу (start → end).
redis.zremrangebyscore(name, min, max)                → Удаляет элементы по score (min ≤ score ≤ max).
redis.zremrangebylex(name, min, max)                  → Удаляет элементы по алфавиту (используется, если ключи — строки, но требует одной длины).
```
Пример:
```python
redis.zadd("leaderboard", {"Alice": 100, "Bob": 200})
redis.zrange("leaderboard", 0, -1, withscores=True)  
# [('Alice', 100.0), ('Bob', 200.0)]
redis.zincrby("leaderboard", "Alice", 50)  
# Теперь Alice: 150
```
Sorted Set отлично подходит для лидербордов, рейтингов и приоритезации задач.

* ## List Blocking Commands / Redis
Блокирующие операции с очередями (Lists) — используются для ожидания появления элементов, если список пуст.
Если вам нужна синхронизация процессов без постоянного опроса сервера, то вот ваши три верных соратника:
```python
redis.blpop(keys, timeout) → Блокирующий LPOP.
Ждет, пока появится элемент в начале списка keys, или возвращает None, если истек timeout.
Пример: «Эй, Redis, дай мне первый элемент!» — «Подожди, он ещё не приготовился…»

redis.brpop(keys, timeout) → Блокирующий RPOP.
Работает так же, но ждет элемент с конца списка.
Идеальный вариант для ленивых — можно взять последний элемент, когда все поели.

redis.brpoplpush(source, destination, timeout) → Блокирующая версия RPOPLPUSH.
Блокирует и ждет, пока появится элемент в source, затем перемещает его в destination (аналог RPOPLPUSH).
Типичный сотрудник офиса: «Мне нечего делать… О, новая задача!»

Параметры:

keys — список ключей (проверяет в порядке, пока не найдет непустой).
timeout — время ожидания (в секундах, 0 — ждать бесконечно).
```
Пример:
```python
redis.blpop("queue", 5)  # Подождёт 5 секунд, если список пуст
redis.brpoplpush("tasks", "processing", 10)  # Ожидание 10 сек, если задач нет
```
Пример использования в очередях:
```python
redis.rpush("queue", "task1", "task2")
redis.blpop("queue", 5)  # Вернёт ('queue', 'task1'), если очередь не пуста
Блокирующие команды полезны для асинхронных очередей задач.
```

* ## Pub/Sub Commands / Redis
Redis Pub/Sub — это механизм публикации и подписки на сообщения в реальном времени. Работает, как радио: одни отправляют сообщения, другие их слушают.
```python
redis.publish(channel, message)  → Отправляет message в channel (если кто-то подписан — он получит его).
redis.subscribe(*channels)       → Подписывается на channels, слушает новые сообщения.
redis.unsubscribe(*channels)     → Отписывается от channels.
redis.psubscribe(*patterns)      → Подписывается на каналы по шаблону (например, "news.*" — все каналы, начинающиеся с news.).
redis.punsubscribe(*patterns)    → Отписывается от шаблонных подписок.
```
Пример:
Публикация:
```python
redis.publish("news.sports", "Футбол сегодня вечером!")
```
Подписчик:
```python
pubsub = redis.pubsub()
pubsub.subscribe("news.sports")
for message in pubsub.listen():
    print(message)  # Будет выводить все новые сообщения из "news.sports"
```
Когда использовать?
Реальное время (чат, уведомления, лайв-трекеры).
Событийная архитектура (например, обновления кеша).
Простая и эффективная система обмена сообщениями без лишних сложностей! 🚀


* ## Transaction Commands / Redis
Транзакции в Redis позволяют выполнять несколько команд атомарно, т. е. либо все выполнятся, либо ни одна.
```python
redis.watch(*keys)   → Следит за keys, если их значение изменится до execute(), транзакция отменится.
redis.unwatch()      → Перестаёт следить за ключами, отменяя эффект watch().
redis.multi()        → Начинает транзакцию (последующие команды не выполняются сразу, а ставятся в очередь).
redis.execute()      → Выполняет все команды внутри multi().
redis.discard()      → Отменяет все команды в multi().
```
Пример использования:
```python
redis.watch("balance")  
current_balance = int(redis.get("balance"))  

if current_balance >= 50:
    pipeline = redis.pipeline()  # Включаем режим транзакции
    pipeline.multi()
    pipeline.decr("balance", 50)
    pipeline.incr("items_bought", 1)
    pipeline.execute()  # Атомарное выполнение
else:
    redis.unwatch()  # Отменяем слежку, если баланс недостаточен
```
🔹 Когда использовать?

Гарантия целостности данных (например, списание денег, обновление статистики).
Исключение состояния гонки (race condition).

* ## Connection Commands / Redis
Эти команды управляют соединением клиента с сервером Redis.
```python
redis.auth(password)  → Аутентифицируется на сервере с password (если Redis настроен на использование пароля).
redis.select(db)      → Переключается на базу данных db (в Redis по умолчанию 16 БД, нумерация с 0).
redis.echo(message)   → Возвращает message (удобно для тестирования соединения).
redis.quit()          → Закрывает соединение с сервером.
```
Пример:
```python
redis.auth("my_secret_password")  
redis.select(1)  
print(redis.echo("Hello Redis!"))  # Выведет: b'Hello Redis!'  
redis.quit()  
```
🔹 Когда использовать?
```python
auth() — если Redis требует пароль.
select() — если используешь несколько баз.
echo() — для тестирования соединения.
quit() — для явного закрытия соединения.
```

* ## Server Commands / Redis
Эти команды управляют состоянием и сохранением данных сервера Redis.
```python
redis.info(section=None) → Выводит информацию о сервере (если section=None — полная инфа, можно указать "memory", "cpu", "stats", "replication" и т. д.).
redis.monitor()          → Показывает в реальном времени все команды, выполняемые на сервере (удобно для отладки).
redis.bgsave()           → Асинхронно сохраняет данные в RDB-файл (без блокировки работы сервера).
redis.bgrewriteaof()     → Оптимизирует AOF (Append Only File), перезаписывая его для уменьшения размера.
```
Пример использования:
```python
print(redis.info("memory"))  # Информация о памяти
redis.bgsave()  # Создаст резервную копию (RDB)
redis.bgrewriteaof()  # Оптимизирует AOF
```

🔹 Когда использовать?
```python
info()         → Диагностика сервера.
monitor()      → Отладка запросов в реальном времени.
bgsave()       → Бэкапы с минимальной нагрузкой.
bgrewriteaof() → Очистка и оптимизация AOF.
```

----
<a id="RDB-AOF"></a>
## RDB (Redis Database) & AOF (Append Only File)
RDB и AOF — это два способа сохранения данных в Redis. Каждый из них используется для обеспечения постоянства данных и восстановления после перезагрузки сервера.

* ### RDB (Redis Database)
Описание:
RDB — это файл снимков базы данных Redis, который создается в определённые моменты времени. Он сохраняет состояние всей базы данных в момент снимка.

#### Как работает:

Redis выполняет полное сохранение состояния в файл .rdb, например, раз в несколько минут или после выполнения определённого числа операций (в зависимости от конфигурации).
Снимки делаются асинхронно, что не блокирует сервер.
#### Преимущества:

Быстрое восстановление данных (т.к. только один файл нужно загрузить).
Подходит для случаев, когда не требуется высокая частота сохранений (например, для бэкапирования).
#### Недостатки:

Потеря данных между снимками (если сервер упал до следующего снимка, изменения не будут сохранены).
### Команды:
```
BGSAVE — Запускает асинхронное сохранение.
SAVE — Выполняет синхронное сохранение (блокирует сервер).
```

* ### Пример 1: Создание снимка вручную
```python
import redis

redis_client = redis.StrictRedis(host='localhost', port=6379, db=0)

# Создаём данные
redis_client.set("key1", "value1")
redis_client.set("key2", "value2")

# Сохранение снимка базы данных (RDB)
redis_client.bgsave()
```
🔹 После выполнения bgsave() Redis создаст файл dump.rdb в папке /var/lib/redis/ (может отличаться).

* ### Пример 2: Автоматическое сохранение через конфиг
Можно настроить автоматическое создание снимков в файле redis.conf:

```ini
save 900 1   # Сохранять, если было хотя бы одно изменение за 900 секунд (15 минут)
save 300 10  # Если за 5 минут было 10 изменений — сохранить
save 60 1000 # Если за 1 минуту было 1000 изменений — сохранить
```
При изменении конфигурации перезапустите Redis:
```bash
redis-cli shutdown
redis-server /etc/redis/redis.conf
```

* ### Пример 3: Восстановление из RDB
Если Redis перезапустить и файл dump.rdb присутствует, база восстановится автоматически.

Можно проверить сохранённые данные:
```python
print(redis_client.get("key1"))  # b'value1'
```


* ### AOF (Append Only File)
Описание:
AOF — это файл, который журналирует каждую операцию записи, выполняемую на сервере Redis (например, добавление или изменение данных). Каждая команда записи сохраняется в этом файле и может быть повторно исполнена для восстановления данных.

#### Как работает:

Каждая запись команд в файл происходит немедленно, что обеспечивает сохранение всех изменений.
Когда Redis перезапускается, он повторно исполняет команды из AOF, чтобы восстановить состояние.
#### Преимущества:

Минимальная потеря данных, т.к. каждое изменение записывается в файл.
Высокая степень безопасности данных, так как можно настроить частоту записи.
#### Недостатки:

Потенциально больший размер файла, т.к. каждое изменение записывается.
Более медленное восстановление данных по сравнению с RDB (нужно выполнять все операции из AOF).
Команды:
```
BGREWRITEAOF — Перезаписывает AOF, уменьшая его размер, удаляя устаревшие данные.
AOF (настройка) — Настройка режима записи (например, always, everysec, no).
```

* ### Пример 1: Включение AOF через конфиг
В файле redis.conf активируйте AOF:

```ini
appendonly yes      # Включаем AOF
appendfsync everysec  # Запись команд в файл каждую секунду
```
Перезапускаем Redis:
```bash
redis-cli shutdown
redis-server /etc/redis/redis.conf
```
Теперь команды будут записываться в appendonly.aof.

* ### Пример 2: Перезапись AOF вручную
```python
# Оптимизация AOF-файла (перезапись для уменьшения размера)
redis_client.bgrewriteaof()
```
Это удалит лишние команды и уменьшит размер appendonly.aof.

* ### Пример 3: Восстановление из AOF
Если Redis запущен с appendonly yes, то при перезапуске он восстановит данные, проиграв все команды из appendonly.aof.


* ### Сравнение RDB и AOF

| Параметр               | RDB                                                        | AOF |
| ---------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------- |
| Тип сохранения         | Снимки (полное состояние базы)                                 | Журнал изменений (по операциям) |
| Частота сохранений     | Устанавливается периодически (например, раз в несколько минут) | Каждое изменение записывается немедленно |
| Производительность     | Быстрое сохранение, минимальная нагрузка                       | Меньше производительность, т.к. каждая запись должна быть сохранена |
| Восстановление данных  | Быстрое восстановление (загрузка одного файла)                 | Медленное восстановление (повторное выполнение операций) |
| Потеря данных          | Потеря данных между снимками                                   | Минимальная потеря данных, в зависимости от настроек |
| Использование          | Для бэкапов или в случаях, когда частота изменений не высока   | Когда важно сохранить все изменения и минимизировать потерю данных |


* ### Комбинированное использование RDB + AOF
Можно использовать RDB и AOF одновременно, чтобы получить преимущества обеих технологий.
RDB обеспечит быстрые снимки базы, а AOF будет сохранять все изменения в реальном времени.
Этот подход предоставляет наилучшую устойчивость к сбоям и минимальную потерю данных.

Чтобы включить оба метода, можно использовать конфигурацию Redis:
* ### Пример 1 : использование RDB + AOF
```bash
save 900 1  # Снимок каждую 1 минуту, если есть хотя бы 1 изменение
appendonly yes  # Включить AOF - Логирование каждой команды
appendfsync everysec  # Записывать изменения каждую секунду
```

* ### Пример 2 : использование RDB + AOF
Можно включить оба механизма в redis.conf:

```ini
save 60 1000  # Снимок раз в минуту при 1000 изменениях
appendonly yes  # Включить AOF - Логирование каждой команды
appendfsync everysec  # Запись команд каждую секунду
```
Такой подход даёт баланс между безопасностью и производительностью.

### Вывод:
Если важна скорость — использовать **RDB**.

Если важна надёжность данных — использовать **AOF**.

Если нужен компромисс — использовать **оба**. 🚀


---
<a id="Examples"></a>
## 🔥 Примеры команд для работы с Redis через библиотеку redis-py:

#### Подключение и настройки
```python
import redis

# Создание подключения к Redis
redis_client = redis.StrictRedis(host='localhost', port=6379, db=0, password=None)
```
#### Работа с ключами
```python
# Установка значения для ключа
redis_client.set('my_key', 'Hello, Redis!')

# Получение значения ключа
value = redis_client.get('my_key')
print(value)  # b'Hello, Redis!'

# Удаление ключа
redis_client.delete('my_key')

# Проверка существования ключа
exists = redis_client.exists('my_key')
print(exists)  # 0 (False)
```

#### Работа с хэшами (Hash)
```python
# Установка значений для хэша
redis_client.hset('my_hash', 'field1', 'value1')
redis_client.hset('my_hash', 'field2', 'value2')

# Получение значения по ключу в хэше
value = redis_client.hget('my_hash', 'field1')
print(value)  # b'value1'

# Получение всех полей и значений хэша
hash_values = redis_client.hgetall('my_hash')
print(hash_values)  # {b'field1': b'value1', b'field2': b'value2'}

# Удаление поля в хэше
redis_client.hdel('my_hash', 'field1')
```

#### Работа со списками (List)
```python
# Добавление элементов в список (справа и слева)
redis_client.rpush('my_list', 'value1', 'value2', 'value3')
redis_client.lpush('my_list', 'value0')

# Получение всех элементов из списка
list_values = redis_client.lrange('my_list', 0, -1)
print(list_values)  # [b'value0', b'value1', b'value2', b'value3']

# Удаление элемента из начала списка
popped_value = redis_client.lpop('my_list')
print(popped_value)  # b'value0'
```

#### Работа с множествами (Set)
```python
# Добавление элементов в множество
redis_client.sadd('my_set', 'apple', 'banana', 'cherry')

# Проверка наличия элемента в множестве
is_member = redis_client.sismember('my_set', 'apple')
print(is_member)  # True

# Получение всех элементов множества
set_members = redis_client.smembers('my_set')
print(set_members)  # {b'apple', b'banana', b'cherry'}

# Удаление элемента из множества
redis_client.srem('my_set', 'banana')
```

#### Работа с отсортированными множествами (Sorted Set)
```python
# Добавление элементов с баллами (score)
redis_client.zadd('my_sorted_set', {'Alice': 100, 'Bob': 200, 'Charlie': 150})

# Получение элементов из отсортированного множества
sorted_set_values = redis_client.zrange('my_sorted_set', 0, -1, withscores=True)
print(sorted_set_values)  # [(b'Alice', 100.0), (b'Charlie', 150.0), (b'Bob', 200.0)]

# Получение элементов с баллом в определенном диапазоне
range_by_score = redis_client.zrangebyscore('my_sorted_set', 100, 150, withscores=True)
print(range_by_score)  # [(b'Alice', 100.0), (b'Charlie', 150.0)]
```

#### Работа с транзакциями (MULTI/EXEC)
```python
# Использование транзакции
pipeline = redis_client.pipeline()

# Помещаем несколько команд в очередь
pipeline.set('key1', 'value1')
pipeline.set('key2', 'value2')
pipeline.incr('counter')

# Выполняем все команды
responses = pipeline.execute()
print(responses)  # [True, True, 1]
```

#### Работа с Pub/Sub (публикация и подписка)
```python
# Подключаемся к каналу
pubsub = redis_client.pubsub()

# Подписка на канал
pubsub.subscribe('my_channel')

# Публикация сообщения
redis_client.publish('my_channel', 'Hello, subscribers!')

# Получаем сообщения
for message in pubsub.listen():
    print(message)  # Вставляется сообщение о новой публикации
```

#### Мониторинг команд
```python
# Мониторинг всех команд Redis
for command in redis_client.monitor():
    print(command)  # Выводит все команды, выполняемые на сервере
```

#### Работа с RDB и AOF
```python
# Асинхронное сохранение базы данных в RDB
redis_client.bgsave()

# Перезапись AOF
redis_client.bgrewriteaof()
```

#### Использование различных баз данных
```python
# Переключение на другую базу данных (по умолчанию 0)
redis_client.select(1)
```
Эти примеры показывают основные операции с Redis через библиотеку redis-py. 

----
<a id="Cases"></a>
## 🔥 Примеры кейсов использования Redis 

* ## 1. Caching
Redis можно использовать для кэширования часто используемых данных, снижая нагрузку на ваше основное хранилище данных. Вот пример того, как реализовать кэширование с помощью Redis в Python
```python
import redis

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

def get_data_from_cache(key):
    # Check if data exists in the cache
    if r.exists(key):
        # Retrieve data from the cache
        data = r.get(key)
        return data.decode('utf-8')  # Convert bytes to string
    else:
        # Fetch data from the primary data source
        data = fetch_data_from_source()

        # Store data in the cache with a timeout of 1 hour
        r.setex(key, 3600, data)
        return data
```
* ## 2. Pub/Sub (Publish/Subscribe):
Redis поддерживает паттерн pub/sub, позволяя вам создавать системы обмена сообщениями. Вот пример:
```python
import redis
import time

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

def publish_message(channel, message):
    # Publish a message to the specified channel
    r.publish(channel, message)

def subscribe_channel(channel):
    # Subscribe to a channel and process incoming messages
    pubsub = r.pubsub()
    pubsub.subscribe(channel)

    for message in pubsub.listen():
        print(message['data'].decode('utf-8'))  # Process the received message
```
* ## 3. Rate Limiting:
Redis можно использовать для реализации ограничения скорости, чтобы контролировать количество запросов или операций за период времени. Пример:
```python
import redis

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

def check_rate_limit(ip_address):
    # Increment the request count for the IP address
    request_count = r.incr(ip_address)

    # If the count exceeds the limit (e.g., 100 requests per minute), deny the request
    if request_count > 100:
        return False

    return True
```
* ## 4. Session Storage:
Redis можно использовать для хранения данных сеанса в веб-приложениях. Пример:
```python
import redis
import uuid

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

def create_session(user_id):
    # Generate a unique session ID
    session_id = str(uuid.uuid4())

    # Store the session data in Redis with a timeout of 30 minutes
    r.setex(session_id, 1800, user_id)

    return session_id

def get_user_id_from_session(session_id):
    # Retrieve the user ID from the session data in Redis
    user_id = r.get(session_id)

    if user_id is not None:
        return user_id.decode('utf-8')  # Convert bytes to string
    else:
        return None
```
* ## 5. Leaderboard:
Redis можно использовать для создания таблиц лидеров или рейтингов на основе набранных баллов. Пример:
```python
import redis

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

def update_score(player_id, score):
    # Update the score of a player
    r.zadd('leaderboard', {player_id: score})

def get_leaderboard():
    # Get the top 10 players from the leaderboard
    leaderboard = r.zrevrange('leaderboard', 0, 9, withscores=True)

    for player, score in leaderboard:
        print(f"Player: {player.decode('utf-8')}, Score: {score}")
```
Это лишь несколько примеров того, как Redis можно использовать в Python. Redis предоставляет множество других мощных функций и структур данных, которые можно использовать в различных приложениях.

<a id="Client"></a>
## ⚡️ Example that demonstrates how to use Redis with the redis-py - popular Redis client for Python.

Redis  – это быстрое хранилище данных типа «ключ‑значение» в памяти с открытым исходным кодом.
Ниже приведен пример, демонстрирующий, как использовать Redis с библиотекой redis-py, которая является популярным клиентом Redis для Python.
```python
$ pip install redis

import redis

# Create a Redis client
r = redis.Redis(host='localhost', port=6379, db=0)

# String Operations
r.set('mykey', 'Hello Redis!')
value = r.get('mykey')
print(value)  # Output: b'Hello Redis!'

# List Operations
r.lpush('mylist', 'element1')
r.lpush('mylist', 'element2')
r.rpush('mylist', 'element3')
elements = r.lrange('mylist', 0, -1)
print(elements)  # Output: [b'element2', b'element1', b'element3']

# Set Operations
r.sadd('myset', 'member1')
r.sadd('myset', 'member2')
r.sadd('myset', 'member3')
members = r.smembers('myset')
print(members)  # Output: {b'member2', b'member1', b'member3'}

# Hash Operations
r.hset('myhash', 'field1', 'value1')
r.hset('myhash', 'field2', 'value2')
value = r.hget('myhash', 'field1')
print(value)  # Output: b'value1'

# Sorted Set Operations
r.zadd('mysortedset', {'member1': 1, 'member2': 2, 'member3': 3})
members = r.zrange('mysortedset', 0, -1)
print(members)  # Output: [b'member1', b'member2', b'member3']

# Delete a key
r.delete('mykey')

# Close the Redis connection
r.close()
```

---

<a id="Vector"></a>
## 🔥 Redis as a Vector Database ⚡️
This case was applied recently in my LLM project with RAG. 

it is possible to use Redis instead of Pinecone or Milvus or Weaviate or ChromaDB, for example.

### Run in docker
It is necessary to use ```redis-stack``` !
```bash
docker run -d --name redis-stack -p 6379:6379 -p 8001:8001 redis/redis-stack:latest redis-server --appendonly yes
```

### Connect to Redis
```python
import redis

# Connect to Redis
redis_client = redis.Redis(
            host=REDIS_HOST,
            port=REDIS_PORT,
            password=REDIS_PASSWORD,
            socket_timeout=30,  # Увеличиваем тайм-аут
            socket_connect_timeout=30  # Увеличиваем тайм-аут соединения
        )
        if self.redis_client.ping():
            print("...Connection successful!")
```
### drop index
```python
# drop index
index_name = INDEX_NAME  
try:
    self.redis_client.ft(index_name).dropindex()  # — удаляет RediSearch-индекс
    print("...Index dropped")
except:
    # Index does not exist
    print("...Index does not exist")
```

### load embeddings to redis 
```python
# load embeddings to redis 
vector_dim = len(embeddings[0]["vector"])  # length of the vectors

# Initial number of vectors
vector_number = len(embeddings)

# Define RediSearch fields
text = TextField(name="text")                                      # определяет текстовое поле
text_embedding = VectorField("vector",                             # определяет векторное поле
                             "FLAT", {
                                 "TYPE": "FLOAT32",                # тип данных (32-битные float)
                                 "DIM": vector_dim,                # размерность векторов
                                 "DISTANCE_METRIC": "COSINE",      # метрика косинусного расстояния
                                 "INITIAL_CAP": vector_number,     # начальная емкость индекса
                             }
                             )
fields = [text, text_embedding]
```

### Create RediSearch Index
```python
# Create RediSearch Index

# Check if index exists
try:
    self.redis_client.ft(INDEX_NAME).info()    # получает информацию об индексе. Если индекса нет, срабатывает except, и создается новый
    print("...Index already exists")
except:
    # Create RediSearch Index
    self.redis_client.ft(INDEX_NAME).create_index(    # создает новый RediSearch-индекс
        fields=fields,
        definition=IndexDefinition(                   
            prefix=[PREFIX],                          # указывает, что индекс будет применяться к ключам, начинающимся с PREFIX
            index_type=IndexType.HASH                 # использует HASH для хранения данных
        )
    )
```

### load embeddings to redis
```python
# load embeddings to redis
for embedding in embeddings:
    key = f"{PREFIX}:{str(embedding['id'])}"              # генерирует ключ для каждого объекта
    embedding["vector"] = np.array(embedding["vector"], dtype=np.float32).tobytes()  # сериализует вектор в bytes
    self.redis_client.hset(key, mapping=embedding)        # сохраняет объект как HASH в Redis
print(
    f"Loaded {self.redis_client.info()['db0']['keys']} documents in Redis search index with name: {INDEX_NAME}")  # получает количество ключей в базе db0

```

### Search vector in database Redis
```python
    def search_redis(self,
                     user_query: str,
                     index_name: str = "embeddings-index",
                     vector_field: str = "vector",
                     return_fields: list = ["text", "vector_score"],
                     hybrid_fields="*",
                     k: int = 5,
                     print_results: bool = False,
                     ):
        # Creates embedding vector from user query
        embedded_query = client.embeddings.create(
            input=user_query,
            model="text-embedding-ada-002"
        ).data[0].embedding
        # Prepare the Query
        base_query = f"{hybrid_fields}=>[KNN {k} @{vector_field} $vector AS vector_score]"  # ищет k ближайших соседей по полю vector_field
        query = (
            Query(base_query)
            .return_fields(*return_fields)            # возвращает указанные поля (text, vector_score)
            .sort_by("vector_score")                  # сортирует по vector_score
            .paging(0, k)                             # задает лимит k
            .dialect(2)        # версия синтаксиса запроса в RediSearch. Если не указать, по умолчанию используется dialect(1)
        )
        params_dict = {
            "vector": np.array(embedded_query).astype(dtype=np.float32).tobytes()
        }
        # perform vector search
        results = self.redis_client.ft(index_name).search(query, params_dict)    # выполняет поиск
        if print_results:
            for i, doc in enumerate(results.docs):
                score = 1 - float(doc.vector_score)
                print(f"...{i}. {doc.text} (Score: {round(score, 3)})")
        return [doc["text"] for doc in results.docs]
```

### Call: 
```python
# keywords - to search
# Get the results
results = search_redis(keywords, print_results=True)
```
Вызывается функция search_redis() :
* Преобразует keywords в векторное представление.
* Запускает поиск ближайших векторов в Redis.
* Выводит результаты с их сходством.

🚀 Этот код использует Redis как векторную базу данных для поиска ближайших соседей!

 Команды Redis, которые используются в коде:

ping() — проверяет соединение.

dropindex() — удаляет индекс.

info() — проверяет существование индекса.

create_index() — создает индекс.

hset() — записывает данные в HASH.

info()['db0']['keys'] — получает количество записей.

search() — выполняет поиск по вектору.


## Что такое индекс в Redis?
В Redis индекс — это структура, которая позволяет быстро находить и фильтровать данные, особенно при использовании RediSearch (модуля полнотекстового и векторного поиска).

### 🔹 Основные особенности индекса в Redis:
Не является традиционным индексом (как в SQL)

В обычном Redis данные хранятся в ключ-значение без индексов.
Поиск возможен только по ключам (keys *) или через хеши, множества и списки.
RediSearch индексирует данные

Позволяет фильтровать, сортировать и искать документы быстрее.
Поддерживает полнотекстовый поиск, поиск по вектору, фильтрацию по полям.
Используется для ускорения поиска

Если просто сохранять JSON или хеши в Redis, сложные запросы будут медленными.
Индексы позволяют искать по тексту, фильтровать по числам, сортировать результаты.

### 🔹 Как создать индекс в Redis? (Пример с RediSearch)
  
#### 1️⃣ Определяем поля (например, текст и векторное представление):

```python
from redis.commands.search.field import TextField, VectorField

text = TextField(name="text")  # Полнотекстовый поиск
vector = VectorField("vector", "FLAT", {  
    "TYPE": "FLOAT32",
    "DIM": 1536,  # Размерность эмбеддинга
    "DISTANCE_METRIC": "COSINE"
})
fields = [text, vector]
```
#### 2️⃣ Создаем индекс

```python
from redis.commands.search.indexDefinition import IndexDefinition, IndexType

INDEX_NAME = "embeddings-index"

redis_client.ft(INDEX_NAME).create_index(
    fields=fields,
    definition=IndexDefinition(
        prefix=["doc:"],  # Префикс ключей, которые будут индексироваться
        index_type=IndexType.HASH
    )
)
```
#### 3️⃣ Добавляем данные

```python
redis_client.hset("doc:1", mapping={"text": "Пример документа", "vector": vector_data})
```
#### 4️⃣ Делаем поиск

```python
query = Query("*").return_fields("text").sort_by("text")
results = redis_client.ft(INDEX_NAME).search(query)
```

### 🔹 Какие типы индексов есть в Redis?

| Тип индекса	| Описание |
|---|---|
| RediSearch (FT Index)	| Полнотекстовый поиск по тексту и векторные индексы |
| Secondary Indexes (ZSET, HASH)	| Используется для поиска и сортировки по конкретным полям |
| Primary Index (ключи Redis)	| Обычные ключи Redis, поиск по keys * (неэффективно) |

### ✅ Вывод:

Индекс в Redis — это структура, помогающая быстро находить данные, особенно в RediSearch (для полнотекстового поиска и поиска по вектору).

## Чем индекс в Redis отличается от обычных данных?

| Фактор	| Обычные данные (ключ-значение)	| Индексы (RediSearch и др.) |
|---|---|---|
| Хранение	 | Данные хранятся как ключ → значение (String, Hash, List, Set, ZSet)	| Дополнительная структура для быстрого поиска
| Поиск	 | Поиск возможен только по ключу (GET key)	| Можно искать по тексту, числам, вектору
| Фильтрация	 | Нет встроенной фильтрации	| Поддерживается (WHERE age > 30)
| Сортировка	 | Требует сортировки в коде или SORT	| Сортировка встроена
| Производительность	 | Быстрое чтение по ключу, но сложные запросы медленные	| Оптимизировано для сложных поисковых запросов
| Использование	 | Кеширование, хранение настроек, очереди	| Поиск по тексту, векторный поиск, аналитика

🔹 Пример разницы
Обычные данные в Redis (без индекса)
python
Копировать
Редактировать
redis_client.hset("user:1", mapping={"name": "Alex", "age": "30"})
redis_client.hset("user:2", mapping={"name": "Bob", "age": "25"})

# Получаем данные только по ключу
user = redis_client.hgetall("user:1")
✅ Быстро, но нельзя искать всех, у кого age > 25.

Индекс в Redis (с RediSearch)
python
Копировать
Редактировать
from redis.commands.search.field import TextField, NumericField
from redis.commands.search.indexDefinition import IndexDefinition, IndexType

fields = [
    TextField(name="name"),
    NumericField(name="age")
]

# Создаем индекс
redis_client.ft("user_index").create_index(fields=fields, definition=IndexDefinition(prefix=["user:"]))

# Теперь можно делать поиск
query = Query("age:[25 35]").return_fields("name", "age")
results = redis_client.ft("user_index").search(query)
✅ Можно искать, фильтровать, сортировать, что невозможно без индекса.

🔹 Вывод
Обычные данные Redis хороши для быстрого чтения по ключу.
Индексы в Redis (например, RediSearch) позволяют делать поиск по полям, фильтрацию, векторный поиск и сортировку.


---
### Contact me: u123@ua.fm
