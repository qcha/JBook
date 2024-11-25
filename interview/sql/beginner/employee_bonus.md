# Бонусы работников

Задача с [LeetCode](https://leetcode.com/problems/employee-bonus/description/).

## Условие

Имеется две таблицы:

```sql
create table employee(
    empid          integer primary key,
    name           text,
    supervisor     integer,
    salary         integer

);

create table bonus(
    empid          integer unique references employee(empid),
    bonus          integer
);

insert into employee(empid, name, supervisor, salary) values (1, 'jonh', 3, 1000);
insert into employee(empid, name, supervisor, salary) values (2, 'dan', 3, 2000);
insert into employee(empid, name, supervisor, salary) values (3, 'brad', null, 4000);
insert into employee(empid, name, supervisor, salary) values (4, 'thomas', 3, 4000);


insert into bonus(empid, bonus) values (2, 500);
insert into bonus(empid, bonus) values (4, 2000);
```

Таблицы показывают связь работника и его премии.

Необходимо написать запрос, который выведет имена и премии всех работников, у которых размер премии был меньше 1000.

### Пример

Input:

Employee table:

| empId | name   | supervisor | salary |
|-------|--------|------------|--------|
| 3     | Brad   | null       | 4000   |
| 1     | John   | 3          | 1000   |
| 2     | Dan    | 3          | 2000   |
| 4     | Thomas | 3          | 4000   |

Bonus table:

| empId | bonus |
|-------|-------|
| 2     | 500   |
| 4     | 2000  |

Output:

| name | bonus |
|------|-------|
| Brad | null  |
| John | null  |
| Dan  | 500   |

## Решение

В данной задаче необходимо сделать join двух таблиц, при этом по условию, необходимо учесть тех работников, у которых вообще не было премии (нет связей в таблице `bonus`), для этого понадобится `left outer join`:

```sql
select 
    e.name,
    b.bonus
from
    employee e
left outer join
    bonus b
on 
    e.empid = b.empid
where
    b.bonus is null 
or
    b.bonus < 1000
```
