# Top Travellers

Задача с [LeetCode](https://leetcode.com/problems/top-travellers/description/).

## Условие

Даны таблицы:

```sql
create table users(
  id int primary key,
  name text
);

create table rides(
  id int primary key,
  user_id int,
  distance int
);
```

Напишите решение для отчёта о расстоянии, которое прошел каждый пользователь.

Таблица результатов должна быть упорядочена по `travelled_distance` в порядке убывания. Если два или более пользователей преодолели одинаковое расстояние, nj отсортируйте их по имени в порядке возрастания.

### Примеры

Input:

Users table:

| id   | name      |
|------|-----------|
| 1    | Alice     |
| 2    | Bob       |
| 3    | Alex      |
| 4    | Donald    |
| 7    | Lee       |
| 13   | Jonathan  |
| 19   | Elvis     |

Rides table:

| id   | user_id  | distance |
|------|----------|----------|
| 1    | 1        | 120      |
| 2    | 2        | 317      |
| 3    | 3        | 222      |
| 4    | 7        | 100      |
| 5    | 13       | 312      |
| 6    | 19       | 50       |
| 7    | 7        | 120      |
| 8    | 19       | 400      |
| 9    | 7        | 230      |

Output:

| name     | travelled_distance |
|----------|--------------------|
| Elvis    | 450                |
| Lee      | 450                |
| Bob      | 317                |
| Jonathan | 312                |
| Alex     | 222                |
| Alice    | 120                |
| Donald   | 0                  |

```sql
create table users(
  id int primary key,
  name text
);

create table rides(
  id int primary key,
  user_id int,
  distance int
);

insert into users
  (id, name) 
values
  (1, 'Alice'),
  (2, 'Bob'),
  (3, 'Alex'),
  (4, 'Donald'),
  (7, 'Lee'),
  (13, 'Jonathan'),
  (19, 'Elvis');

insert into rides 
  (id, user_id, distance) 
values
  (1, 1, 120),
  (2, 2, 317),
  (3, 3, 222),
  (4, 7, 100),
  (5, 13, 312),
  (6, 19, 50),
  (7, 7, 120),
  (8, 19, 400),
  (9, 7, 230);
```

## Решение

Не у всех пользователей могут быть походы/поездки, а значит нам нужен `left join`. После необходимо сгруппировать пользователей и у агрегатов применить `sum` на дистанцию.

Важно отметить, что надо применить `coalesce`, так как иначе `null` дистанции не войдут в выборку.

```sql
select
    u.name,
    coalesce(sum(r.distance), 0) as travelled_distance
from 
    users u
left join 
    rides r 
on 
  u.id = r.user_id
group by
    u.id, u.name
order by 
    travelled_distance desc,
    u.name asc;
```
