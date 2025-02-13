# Department Highest Salary

Задача с [LeetCode](https://leetcode.com/problems/department-highest-salary/).

## Условие

Даны таблицы:

```sql
create table employee(
  id int primary key,
  name text,
  salary int,
  department_id int
);

create table department(
  id int primary key,
  name text
);
```

Каждая строка таблицы `employee` указывает идентификатор, имя, зарплату сотрудника и содержит идентификатор его отдела.
Гарантируется, что таблицы заполнены данными и в них отсутствуют `null` значения.

Напишите решение для поиска сотрудников с самой высокой зарплатой в каждом из отделов.

### Примеры

Input:

Employee table:

| id | name  | salary | department_id |
|----|-------|--------|---------------|
| 1  | Joe   | 70000  | 1             |
| 2  | Jim   | 90000  | 1             |
| 3  | Henry | 80000  | 2             |
| 4  | Sam   | 60000  | 2             |
| 5  | Max   | 90000  | 1             |

Department table:

| id | name  |
|----|-------|
| 1  | IT    |
| 2  | Sales |

Output:

| Department | Employee | Salary |
|------------|----------|--------|
| IT         | Jim      | 90000  |
| Sales      | Henry    | 80000  |
| IT         | Max      | 90000  |

```sql
create table employee(
  id int primary key,
  name text,
  salary int,
  department_id int
);

create table department(
  id int primary key,
  name text
);

insert into department(id, name) values(1, 'IT');
insert into department(id, name) values(2, 'Sales');

insert into employee(id, name, salary, department_id) values(1, 'Joe', 70000, 1);
insert into employee(id, name, salary, department_id) values(2, 'Jim', 90000, 1);
insert into employee(id, name, salary, department_id) values(3, 'Max', 90000, 1);

insert into employee(id, name, salary, department_id) values(4, 'Sam', 60000, 2);
insert into employee(id, name, salary, department_id) values(5, 'Henry', 80000, 2);
```

## Решение

### CTE

В самом начале нам надо получить связь отдел-максимальная зарплата в нем.
Для этого создадим CTE, где сгруппируем по идентификатору отдела и найдем максимальную зарплату в нем.

```sql
with tmp as (
    select 
        e.department_id, 
        max(e.salary) as salary
    from
        employee e
    group by 
        e.department_id
)
```

После этого найдем всех работников с такими идентификаторами и зарплатами как в CTE:

```sql
select 
    d.name as "Department",
    e.name as "Employee",
    e.salary as "Salary" 
from
    employee e
inner join 
    department d
on 
    e.department_id = d.id
where 
    (d.id, e.salary)
in (
        select 
            department_id,
            salary
        from
            tmp
    )
```

### ALL

Еще одним решением является использование `all`.

Стандартным `join`-ом получим сотрудников и названия отделов, но добавим условие фильтрации:

```sql
where
    e.salary >= all(
        select
            salary
        from
            employee
        where
            department_id = e.department_id
    )
```

Оператор `ALL` проверяет, удовлетворяют ли условию все элементы списка.

Т.е. зарплата сотрудника должна быть больше или равна зарплат всех сотрудников из подзапроса. В подзапросе же получим всех сотрудников отдела.

Итоговый запрос:

```sql
select 
    d.name as "Department",
    e.name as "Employee",
    e.salary as "Salary" 
from
    employee e
join
    department d 
on 
    d.id = e.department_id
where
    e.salary >= all(
        select
            salary
        from
            employee
        where
            department_id = e.department_id
    )
```

### RANK

Третьим решением является использование оконных функций, в данном случае `RANK()`.

Оконные функции (не по определению, но на практике чаще всего) применяются на партицированные данные.

Поэтому, выведем все необходимые для результата поля, а также добавим новое с элиасом `rnk`, в котором функция `RANK()` применяется на партицию по отделу с отсортированными в убывающем порядке зарплатами.

```sql
select 
    d.name as department,
    e.name as employee,
    e.salary as salary,
    rank() over (partition by d.id order by e.salary desc) as rnk
from
  employee e
join
  department d
on
  d.id = e.departmentId;
```

Как результат, в поле `rnk` значение 1 будет присвоено только тем работникам, зарплаты которых соответствуют максимальному значению зарплат по отделу (причем неважно, один ли работник получает максимальную зарплату в отделе, или несколько; если их несколько - у каждого поле `rnk` получит значение 1).

Остается только отфильтровать записи, где `rnk = 1`.

Итоговый запрос:

```sql
with cte as (
  select 
      d.name as department,
      e.name as employee,
      e.salary as salary,
      rank() over (partition by d.id order by e.salary desc) as rnk
  from
    employee e
  join
    department d
  on
    d.id = e.departmentId
)

select
  department,
  employee,
  salary
from
  cte
where
  rnk = 1;
```
