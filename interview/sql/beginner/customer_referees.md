# Найти рекомендателя клиента

Задача с [LeetCode](https://leetcode.com/problems/find-customer-referee/description/).

## Условие

Имеется таблица:

```sql
create table customer(
    id          integer primary key,
    name        text,
    referee_id  integer
);

insert into customer (id, name, referee_id) values (1, 'Zack', null);
insert into customer (id, name, referee_id) values (2, `Alex`, null);
insert into customer (id, name, referee_id) values (3, `Mark`, 2);
```

Таблица отображает пользователей и рекомендации от пользователей (столбец `referee_id`).

Напишите запрос, чтобы с помощью него получить все имена пользоватлей, которые **не были** порекомендованы пользователем с идентификатором 2.

### Пример

Input:

Таблица `customer`:

| id | name    | referee_id |
|----|---------|------------|
| 1  | `Zack`  |     null   |
| 2  | `Alex`  |     null   |
| 3  | `Mark`  |      2     |

Output:

| id | email   |
|----|---------|
| 1  | `Zack`  |
| 2  | `Alex`  |

## Решение

В данной задаче необходимо не забыть значения с `null` в столбце `referee_id`, в остальном же решается простым условием в `where`:

```sql
select
    name
from
    customer
where 
    referee_id != 2 
or
    referee_id is null
```
