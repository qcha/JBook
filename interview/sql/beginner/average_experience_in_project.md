# Средний опыт сотрудников на проекте

Задача с [LeetCode](https://leetcode.com/problems/project-employees-i/description/).

## Условие

Даны следующие таблицы:

```sql
create table project(
  project_id int,
  employee_id int
);

create table employee(
  employee_id int primary key,
  name text,
  experience_years int
);

insert into project(project_id, employee_id) values (1, 1);
insert into project(project_id, employee_id) values (1, 2);
insert into project(project_id, employee_id) values (1, 3);
insert into project(project_id, employee_id) values (2, 1);
insert into project(project_id, employee_id) values (2, 4);

insert into employee(employee_id, name, experience_years) values (1, 'Khaled', 3);
insert into employee(employee_id, name, experience_years) values (2, 'Ali', 2);
insert into employee(employee_id, name, experience_years) values (3, 'Jonh', 1);
insert into employee(employee_id, name, experience_years) values (4, 'Doe', 2);
```

Напишите запрос, который вернет средний опыт всех сотрудников для каждого проекта, с точностью до 2 цифр.

Результат можно вывести в любом порядке.

### Примеры

Input:

Project table:

| project_id  | employee_id |
|-------------|-------------|
|      1      |      1      |
|      1      |      2      |
|      1      |      3      |
|      2      |      1      |
|      2      |      4      |

Employee table:

| employee_id | name   | experience_years |
|-------------|--------|------------------|
|      1      | Khaled |        3         |
|      2      | Ali    |        2         |
|      3      | John   |        1         |
|      4      | Doe    |        2         |

Output:

| project_id  | average_years |
|-------------|---------------|
| 1           | 2.00          |
| 2           | 2.50          |

## Решение

Здесь необходимо соединить две таблицы: проектов и сотрудников, после чего необходимо сгруппировать их по отделам. Полученые агрегаты содержат все необходимые данные, останется только найти среднее по опыту в отделе и округлить. Среднее ищем с помощью агрегатной функции `avg`.

В результате:

```sql
select 
    p.project_id,
    round(avg(e.experience_years), 2) as average_years
from
    project p
inner join
    employee e
on
    p.employee_id = e.employee_id
group by
    p.project_id
```
