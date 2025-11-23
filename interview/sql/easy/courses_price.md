# Общая стоимость курсов для студентов

Задача с [Youtube](https://youtu.be/kI6N79Q0wU4?t=282).

*Важно: обратите внимание на то, что в видео на ютубе задачу решили неправильно!*

## Условие

Имеются следующие таблицы:

```sql
create table students(
  id int primary key,
  name text
);
  
create table students_courses(
    student_id int,
    course_id  int
);
  
  
create table courses(
    id int primary key,
    name text,
    price int
);
  
insert into 
    students(id, name) 
values 
    (1, 'Aleksandr'),
    (2, 'Aleksandr'),
    (3, 'Egor'),
    (4, 'Kira'),
    (5, 'Kira');
  
insert into 
    courses(id, name, price) 
values 
    (1, 'C++', 2000),
    (2, 'Go', 10000),
    (3, 'Bred', 5000),
    (4, 'Java', 9000),
    (5, 'JavaScript', 1500);
    
    
    
insert into 
    students_courses(student_id, course_id) 
values 
    (1, 1),
    (1, 2),
    (2, 4),
    (4, 3),
    (4, 4);
```

Необходимо получить имена студентов и общую сумму курсов, которые они посещают, но только для тех студентов, у которых общая стоимость курсов превышает 10_000.

## Решение

Решение заключается в том, что необходимо сгруппировать студентов по id и найти тех, у кого сумма стоимости курсов больше искомой.

```sql
select 
    s.name, 
    sum(c.price)
from
    students s
inner join
    students_courses sc
on
    s.id = sc.student_id
inner join
    courses c
on
    sc.course_id = c.id
group by
    s.id, s.name
having 
    sum(c.price) > 10000;
```

Самая распространенная ошибка заключается в том, что группировка идет по имени студента (чтобы ее использовать в select). Но имена не уникальны и это может привести к ошибке.

Важно отметить, что в некоторых БД(например, `PostgreSQL`) сработает также и подобный запрос:

```sql
select s.name, sum(c.price)
from
    students s
inner join
    students_courses sc
on
    s.id = sc.student_id
inner join
    courses c
on
    sc.course_id = c.id
group by
    s.id
having sum(c.price) > 10000;
```

Так как s.id - это *primary key*.

В `PostgreSQL` есть механизм functional dependency, который разрешает выбирать столбцы, функционально зависящие от тех, что в group by.
