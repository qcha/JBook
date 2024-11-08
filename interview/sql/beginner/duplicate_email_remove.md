# Удалить дубликаты email-ов

Задача с [LeetCode](https://leetcode.com/problems/delete-duplicate-emails/description/).
Также встречается разобранной на YouTube: [Удаление дубликатов email | Практика по SQL](https://www.youtube.com/watch?v=nF-l3tZovKY).

## Условие

Имеется таблица:

```sql
create table person(
    id    integer primary key,
    email text
);

insert into person (id, email) values (1, `john@example.com`);
insert into person (id, email) values (2, `bob@example.com`);
insert into person (id, email) values (3, `john@example.com`);
```

Напишите решение для удаления всех дубликатов писем, оставив только одно уникальное письмо с наименьшим идентификатором.

Следует обратить внимание на то, что вы должны написать оператор `delete`, а не `select`.

### Пример

Input:

Таблица `person`:

| id | email              |
|----|--------------------|
| 1  | `john@example.com` |
| 2  | `bob@example.com`  |
| 3  | `john@example.com` |

Output:

| id | email              |
|----|--------------------|
| 1  | `john@example.com` |
| 2  | `bob@example.com`  |

Объяснение: `john@example.com` встречается дважды, оставляем единственную запись с наименьшим `id` = 1.

## Решение

Если сгруппировать по `email` и найти минимальный `id`, то это и будет именна та строчка, которая должна остаться. Остальные можно будет удалить.

Найдем `id`, которые надо оставить запросом:

```sql
select min(id)
from person
group by email
```

Удалим все `id`, которые не выше найденные:

```sql
delete 
from person
where id not in (
            select min(id)
            from person
            group by email
        )
```

Это и будет решением!

В `MySQL` использовать ту же таблицу в подзапросе будет нельзя:

```text
You can't specify target table 'person' for update in FROM clause
```

Поэтому надо использовать или self-join:

```sql
delete p1
from 
    person p1
inner join
    person p2
on
    p1.email = p2.email
WHERE
    p1.id > p2.id
```

Или сделать из подзапроса выборку и из нее уже удалять:

```sql
delete from person as p1 where p1.id not in (
    select
        *
    from
        (
            select min(id) 
            from person
            group by email
        ) as p2
)
```
