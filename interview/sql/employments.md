# Сотрудники в отделе

Задачи подобного плана показывают то, как человек умеет работать с `sql`, понимает ли как работают агрегатные функции и `join`-ы.

Данную задачу подсмотрел с собеседования на `Middle Python` разработчика на `Youtube`.

## Условие

Имеются следующие таблицы:

```sql
-- Department table
create table departments
(
    id    integer primary key,
    title text
);

insert into departments (id, title)
values (1, 'operational'),
       (2, 'financional'),
       (3, 'it'),
       (4, 'security'),
       (5, 'navvy');

-- Employees table
create table employments
(
    id            serial primary key,
    name          text,
    salary        bigint,
    id_department integer
);

insert into employments
values (1, 'Cora Rowe MD', 50622, 4),
       (2, 'Dario Thiel', 31520, 2),
       (3, 'Mathilde Ondricka', 46322, 4),
       (4, 'Mr. Gerald Champlin', 22758, 3),
       (5, 'Marlee Rolfson', 41965, 3),
       (6, 'Tre Rath', 23140, 3),
       (7, 'Winnifred Rohan', 52989, 2),
       (8, 'Josianne Roberts IV', 20178, 2),
       (9, 'Lucie Maggio', 22238, 4),
       (10, 'Dr. Rae Graham', 44943, 4),
       (11, 'Elvis Franecki', 46914, 2),
       (12, 'Marcelle Berge', 30452, 1)
```

Напишите запрос, который выведет название отделов, где сотрудников больше двух.

## Решение

```sql
select d.title, count(e.id_department)
from employments e inner join departments d
    on e.id_department = d.id
group by e.id_department
having count(e.id_department) > 2
```

Все что нужно в данной задаче - это правильно соединить таблицы, агрегировать полученные данные и применить `count`.

Здесь подойдет простой `inner join`, группировать будем по `id_department`, по департаментам.
Ну а далее останется просто посчитать.
