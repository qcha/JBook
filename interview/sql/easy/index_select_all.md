# Index в where

## Условие

Пусть есть таблица постов:

```sql
create table post (
    id serial primary key,
    owner_id integer not null,
    likes_count integer not null,
    title text,
    content text
);
```

На таблице постов существует следующий индекс:

```sql
create index on post (owner_id)
```

Будет ли попадаение в индекс при выполнении подобного запроса:

```sql
select
    * 
from
    post
where
    likes_count > ? 
and 
    owner_id = ?
```

## Решение

Сделаем explain analyze:

```sql
Bitmap Heap Scan on post (cost=7.11..750.33 rows=124 width=84) (actual time=0.159..0.860 rows=812 loops=1)
Recheck Cond: (owner_id = 42)
Filter: (likes_count > 100)
Rows Removed by Filter: 195
Heap Blocks: exact=657
-> Bitmap Index Scan on idx_post_owner_id (cost=0.00..7.08 rows=371 width=0) (actual time=0.084..0.085 rows=1007 loops=1)
Index Cond: (owner_id = 42)
Planning Time: 1.056 ms
Execution Time: 0.906 ms
```

Как видно, индекс используется для построения битмап карты, потом по карте достаются данные и фильтруются по `likes_count`.

Если же мы попробуем изменить индекс на:

```sql
create index on post (owner_id, likes_count)
```

В таком случае:

```sql
Bitmap Heap Scan on post (cost=5.56..357.18 rows=124 width=84) (actual time=0.168..0.819 rows=784 loops=1)
Recheck Cond: ((owner_id = 42) AND (likes_count > 100))
Heap Blocks: exact=546
-> Bitmap Index Scan on idx_post_owner_likes_covering (cost=0.00..5.53 rows=124 width=0) (actual time=0.099..0.099 rows=784 loops=1)
Index Cond: ((owner_id = 42) AND (likes_count > 100))
Planning Time: 0.152 ms
Execution Time: 0.866 ms
```

Теперь индекс используется по обеим полям сразу и записи не фильтруются.

Попадания в `Index Only Scan` тут также быть не может, так как используется `*`: приходится прочитать все поля строки.

Однако, второй запрос быстрее отрабатывает, так как не используется филтрация, строится более оптимальный `Bitmap Index Scan`.
