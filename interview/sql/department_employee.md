# Сотрудники в отделе

Задачи подобного плана показывают то, как человек умеет работать с `sql`, понимает ли как работают агрегатные функции и `join`-ы.

## Условие

Имеются следующие таблицы:

```sql
-- Department table
create table department
(
    id    integer primary key,
    title text
);

insert into department (id, title)
values (1, 'operational'),
       (2, 'financional'),
       (3, 'it'),
       (4, 'security'),
       (5, 'navvy');

-- Employees table
create table employee
(
    id            serial primary key,
    name          text,
    rank          integer,
    salary        bigint,
    id_department integer,
    status        text
);

insert into employee
values (1, 'Cora Rowe MD', 1, 50622, 4, 'capable'),
       (2, 'Dario Thiel', 6, 31520, 2, 'sick'),
       (3, 'Mathilde Ondricka', 8, 46322, 4, 'sick'),
       (4, 'Mr. Gerald Champlin', 7, 22758, 3, 'vacation'),
       (5, 'Marlee Rolfson', 2, 41965, 3, 'vacation'),
       (6, 'Tre Rath', 6, 23140, 3, 'sick'),
       (7, 'Winnifred Rohan', 3, 52989, 2, 'fired'),
       (8, 'Josianne Roberts IV', 4, 20178, 2, 'fired'),
       (9, 'Lucie Maggio', 1, 22238, 4, 'capable'),
       (10, 'Dr. Rae Graham', 6, 44943, 4, 'vacation'),
       (11, 'Elvis Franecki', 2, 46914, 2, 'sick'),
       (12, 'Marcelle Berge', 5, 30452, 1, 'vacation'),
       (13, 'Dr. Missouri Upton', 5, 35531, 4, 'fired'),
       (14, 'Maximus Kuvalis', 4, 38117, 1, 'vacation'),
       (15, 'Mrs. Norene Huel MD', 6, 8960, 3, 'sick'),
       (16, 'Marisa Dietrich', 3, 2354, 1, 'vacation'),
       (17, 'Mr. Alexandro Jacobi Sr.', 4, 28454, 4, 'fired'),
       (18, 'Malika Aufderhar', 6, 20395, 4, 'fired'),
       (19, 'Ahmed McCullough', 1, 3611, 3, 'capable'),
       (20, 'Liza Considine DVM', 2, 43918, 3, 'vacation'),
       (21, 'Dewayne Schowalter', 6, 49029, 2, 'vacation'),
       (22, 'Miss Isabella DuBuque III', 2, 22277, 4, 'sick'),
       (23, 'Robin Von', 4, 35767, 3, 'vacation'),
       (24, 'Ms. Aylin Gutkowski IV', 8, 43057, 4, 'sick'),
       (25, 'Domenico Bashirian', 5, 42976, 4, 'capable'),
       (26, 'Dr. Nakia Brown', 6, 10425, 1, 'sick'),
       (27, 'Prof. Domingo Veum MD', 2, 24438, 1, 'sick'),
       (28, 'Tremaine Kemmer Jr.', 6, 9322, 3, 'sick'),
       (29, 'Mr. Jimmie Emard MD', 3, 52047, 2, 'capable'),
       (30, 'Carolanne Schinner', 7, 44975, 3, 'capable');
```

Необходимо для каждого отдела посчитать количество сотрудников в статусе != 'capable', максимальный ранг таких сотрудников в отделе, среднюю зарплату сотрудников в отделе, список отсортировать по наименованию отдела.

### Условие задачи о звездочкой

Необходимо для каждого отдела посчитать количество сотрудников в статусе != 'capable', максимальный ранг сотрудника (как с 'capable' статусом, так и нет) в отделе, среднюю зарплату сотрудника (как с 'capable' статусом, так и нет) в отделе, список отсортировать по наименованию отдела и вывести id отдела.

## Решение

```sql
select dep.id, count(dep.id), max(emp.rank), avg(emp.salary)
from employee emp 
right join department dep 
on emp.id_department = dep.id
where emp.status != 'capable'
group by dep.id
order by dep.title
```

Для начала нам надо связать сотрудников с их отделами, для этого мы воспользуемся `right join` к департаментам (ведь могут быть отделы без сотрудников).

Так как нас интересуют только сотрудники с определенным статусом, отфильтруем их через `where`. Далее сгруппируем сотруников по отделам, отсортируем по названию отдела и применим агрегатные функции `count`, `max` и `avg` к полученным записям.

### Решение задачи со звездочкой

В этой задаче нам требуется посчитать количество сотрудников с определнным статусом, а максимальный ранг и среднюю зарплату посчитать для сотрудников с любым рангом.

```sql
select dep.id, count(CASE WHEN emp.status != 'capable' THEN dep.id ELSE NULL END), max(emp.rank), avg(emp.salary)
from employee emp 
right join department dep 
on emp.id_department = dep.id
group by dep.id
order by dep.title
```
