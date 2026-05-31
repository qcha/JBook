# Количество задач сотрудника

Задача с [Youtube](https://youtu.be/6WI_rsiD8Jw?t=3757) как задача с собеседования.

## Условие

Имееются таблицы:

```sql
create table employee(
    id      int primary key,
    name    text,
    rating  int
);

create table task(
    id          int primary key,
    assigne_id  int,
    rating      int
);

insert into
    employee (id, name, rating)
values
    (1, 'Алексей', 5),
    (2, 'Мария',   4),
    (3, 'Дмитрий', 3),
    (4, 'Ольга',   5),
    (5, 'Иван',    2);

insert into 
    task (id, assigne_id, rating)
values
    (1,  1, 3),
    (2,  1, 4),
    (3,  1, 5),  -- у Алексея 3 задачи — не попадает
    (4,  2, 2),
    (5,  2, 4),  -- у Марии 2 задачи — попадает
    (6,  3, 1),  -- у Дмитрия 1 задача — попадает
    (7,  4, 3),
    (8,  4, 4),
    (9,  4, 5),
    (10, 4, 2);  -- у Ольги 4 задачи — не попадает
                 -- у Ивана 0 задач — попадает
```

Необходимо посчитать количесто сотрудников у кого в работе менее трех задач.

## Решение

Сначала отбираем сотрудников с менее чем 3 задачами:

* left join — чтобы сотрудники без задач тоже попали в выборку
* group by e.id — группируем по каждому сотруднику
* count(t.id) — считаем количество задач
* having count(t.id) < 3 — оставляем только тех, у кого задач меньше трёх

После этого считаем количество таких сотрудников.

```sql
with cte as (
    select
        e.id
    from
        employee e
    left join
        task t
    on
        e.id = t.assigne_id
    group by
        e.id
    having
        count(t.id) < 3
  )

select
    count(*) as employee_count
from
    cte;
```

Можно было бы решить и через подзапрос явный - было бы то же самое.
