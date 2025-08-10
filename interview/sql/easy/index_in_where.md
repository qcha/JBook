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
    owner_id 
from
    post
where
    likes_count > ? 
and 
    owner_id = ?
```

## Решение

Полного, Index Only Scan, не будет, так как скорее всего не получится применить условие `likes_count > 100` на уровне индекса, так как likes_count не входит в индекс.
В этом случае вероятнее будет `Bitmap Heap Scan` и далее фильтрация по `likes_count`.

А вот если создать покрывающий индекс:

```sql
create index on post (owner_id, likes_count);
```

Тогда оба поля были бы в индексе и был бы `Index Only Scan`.
