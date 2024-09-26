# Посты от знаменитостей

## Условие

Даны таблицы, описывающие простую социальную сеть:

```sql
create table profile(
    id            bigserial primary key,
    nickname      varchar,
    registered_at timestamp
);

create table post(
    id          bigserial primary key,
    owner_id    bigint referenced profile (id),
    body        text,
    inserted_at timestamp,
    likes_count int -- количество лайков
);

create table subscription_count(
    profile_id      bigint referenced profile (id) unique,
    followers_count int, -- количество подписчиков
    following_count int  -- количество подписок
);
```

Необходимо вывести все посты, опубликованные пользователями, у которых количество подписчиков больше 500.

## Решение

Основная сложность, на мой взгляд, в том, чтобы проанализировать вводные и понять, что для решения на самом деле не нужна таблица `profile`. Это дано скорее для того, чтобы смутить и сбить с толку.

```sql
select 
    post.id
from
    subscription_count
inner join
    post
on 
    subscription_count.profile_id = post.owner_id
where 
    subscription_count.followers_count > 500
```
