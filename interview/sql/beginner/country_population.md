# Население стран

## Условие

Даны таблицы:

```sql
create table country(
  id integer primary key,
  name text
);

create table town(
  id integer primary key,
  name text,
  population integer,
  country_id integer references country(id)
);
```

Предоставляющие связь между населением городов и странами, к которым относятся города.

Необходимо написать запрос, выводящий название и население первых 10 стран с населением более миллиона.

Вывод сделать в обратном порядке, от большего к меньнешму.

## Решение

Для получения связи стран и городов необходимо сделать `join`, после чего сгруппировать страны и посчитать через агрегатную функцию население.

Фильтрация агрегата происходит через `having`, после чего вывести на экран первые 10 в обратном порядке.

```sql
select 
    c.name,
    sum(t.population) as population
from
    country c
inner join
    town t
on
    c.id = t.country_id
group by
    c.name
having  
    sum(t.population) > 1_000_000
order by
    population
desc
limit 
    10
```
