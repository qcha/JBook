# Класс с более чем пятью студентами

Задача с [LeetCode](https://leetcode.com/problems/classes-more-than-5-students/).

## Условие

Имеется таблица:

```sql
create table courses(
  student text,
  "class"   text,
  
  primary key(student, "class")
 );
 
 
insert into courses(student, "class") values ('A', 'Math');
insert into courses(student, "class") values ('A', 'English');
insert into courses(student, "class") values ('A', 'Biology');
insert into courses(student, "class") values ('B', 'Math');
insert into courses(student, "class") values ('B', 'English');
insert into courses(student, "class") values ('C', 'Math');
insert into courses(student, "class") values ('C', 'Computer');
insert into courses(student, "class") values ('C', 'Biology');
insert into courses(student, "class") values ('D', 'Math');
insert into courses(student, "class") values ('E', 'Math');
insert into courses(student, "class") values ('F', 'Math');
```

Напишите запрос, чтобы найти все классы, в которых есть не менее пяти учеников.

Верните таблицу результатов в любом порядке.

### Пример

Input:

| student | class    |
|---------|----------|
| A       | Math     |
| B       | English  |
| C       | Math     |
| D       | Biology  |
| E       | Math     |
| F       | Computer |
| G       | Math     |
| H       | Math     |
| I       | Math     |

Output:

| class   |
|---------|
| Math    |

## Решение

Для решения необходимо сгруппировать учеников по классам и отфильтровать. Фильтрацию производим через `having`, так как работаем с агрегатами и агрегатной функцией `count`.

```sql
select
    class
from
    courses
group by 
    class
having
    count(student) > 5
```
