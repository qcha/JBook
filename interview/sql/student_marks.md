# Студенты и оценки

Задача взята с [открытого собеседования](https://www.youtube.com/watch?v=yEMW2FRkhOo).

Обратите внимание: в видео много опечаток и последнее решение через `left join` неверное!

## Условие

В таблице:

```sql
create table register
(
    student_id    integer,
    mark          integer
);
```

Представлены студенты (идентификаторы) и их оценки.

Создадим и заполним таблицу данными:

```sql
create table register(student_id integer, mark integer);

# 11 троек и 2 пятерки
insert into register(student_id, mark) values (1, 5);
insert into register(student_id, mark) values (1, 3);
insert into register(student_id, mark) values (1, 3);
insert into register(student_id, mark) values (1, 3);
insert into register(student_id, mark) values (1, 3);
insert into register(student_id, mark) values (1, 3);
insert into register(student_id, mark) values (1, 3);
insert into register(student_id, mark) values (1, 3);
insert into register(student_id, mark) values (1, 3);
insert into register(student_id, mark) values (1, 3);
insert into register(student_id, mark) values (1, 3);
insert into register(student_id, mark) values (1, 3);
insert into register(student_id, mark) values (1, 5);

# 9 троек и 4 пятерки
insert into register(student_id, mark) values (2, 5);
insert into register(student_id, mark) values (2, 3);
insert into register(student_id, mark) values (2, 3);
insert into register(student_id, mark) values (2, 3);
insert into register(student_id, mark) values (2, 3);
insert into register(student_id, mark) values (2, 3);
insert into register(student_id, mark) values (2, 3);
insert into register(student_id, mark) values (2, 3);
insert into register(student_id, mark) values (2, 3);
insert into register(student_id, mark) values (2, 3);
insert into register(student_id, mark) values (2, 5);
insert into register(student_id, mark) values (2, 5);
insert into register(student_id, mark) values (2, 5);


# 7 пятерок и 6 четверок
insert into register(student_id, mark) values (3, 5);
insert into register(student_id, mark) values (3, 5);
insert into register(student_id, mark) values (3, 4);
insert into register(student_id, mark) values (3, 4);
insert into register(student_id, mark) values (3, 4);
insert into register(student_id, mark) values (3, 4);
insert into register(student_id, mark) values (3, 5);
insert into register(student_id, mark) values (3, 4);
insert into register(student_id, mark) values (3, 5);
insert into register(student_id, mark) values (3, 4);
insert into register(student_id, mark) values (3, 5);
insert into register(student_id, mark) values (3, 5);
insert into register(student_id, mark) values (3, 5);


# 8 троек и 5 четверок
insert into register(student_id, mark) values (4, 4);
insert into register(student_id, mark) values (4, 4);
insert into register(student_id, mark) values (4, 4);
insert into register(student_id, mark) values (4, 3);
insert into register(student_id, mark) values (4, 3);
insert into register(student_id, mark) values (4, 3);
insert into register(student_id, mark) values (4, 3);
insert into register(student_id, mark) values (4, 3);
insert into register(student_id, mark) values (4, 3);
insert into register(student_id, mark) values (4, 3);
insert into register(student_id, mark) values (4, 3);
insert into register(student_id, mark) values (4, 4);
insert into register(student_id, mark) values (4, 4);

# 8 четверок, 2 тройки и 3 пятерки
insert into register(student_id, mark) values (5, 4);
insert into register(student_id, mark) values (5, 4);
insert into register(student_id, mark) values (5, 4);
insert into register(student_id, mark) values (5, 4);
insert into register(student_id, mark) values (5, 4);
insert into register(student_id, mark) values (5, 4);
insert into register(student_id, mark) values (5, 4);
insert into register(student_id, mark) values (5, 4);
insert into register(student_id, mark) values (5, 3);
insert into register(student_id, mark) values (5, 3);
insert into register(student_id, mark) values (5, 5);
insert into register(student_id, mark) values (5, 5);
insert into register(student_id, mark) values (5, 5);
```

Напишите запрос, который выведет количество пятерок у студентов, у которых количество троек меньше десяти.

## Решение

### Условные выражения

Сгруппировав студентов по id необходимо отфильтровать всех, у кого троек больше 10. Основной подвох здесь в том, что если написать:

```sql
select student_id
from register
where mark = 3
group by student_id
having count(mark) < 10
```

Это будет **ошибкой**!

Ведь в таком случае будут отфильтрованы как те, у кого более 10 троек, так и те, у кого троек **нет вообще**. А это не подходит по условию!

Здесь необходимо вспомнить про [условные выражения](https://postgrespro.ru/docs/postgresql/16/functions-conditional) в запросах:

```sql
select student_id, count(case when mark = 5 then 1 end)
from register
group by student_id
having count(case when mark = 3 then 1 end) < 10
```

Либо сделать выборку через подзапрос (будет работать медленнее):

```sql
select student_id, cnt_5 
from (
  select student_id, count(case when mark = 3 then 1 end) as cnt_3, count(case when mark = 5 then 1 end) as cnt_5
  from register
  group by student_id
) tmp
where cnt_3 < 10
```

### Left Join

Еще одним решением (но, на мой взгляд, запутанным) может быть решение через `left join`.

В подзапросе выделим всех у кого вообще есть тройки и посчитаем их количество.

Далее через `left join` объеденим исходную таблицу и подзапрос. После объединения получается таблица, у которой необходимо отфитльровать всех, у кого количество троек больше 10 (ищем среди трочников тех, у кого их не так много по условию) или `student_id` не `null` (это те самые люди, у которых троек нет вообще).

После этого группируем и получаем результат:

```sql
select register.student_id, count(case when register.mark = 5 then 1 end) as count_of_rows
from register
    left join
        (select student_id, count(student_id) as thirds 
         from register
         where mark = 3
         group by student_id) as subquery 
    on register.student_id = subquery.student_id
where (subquery.thirds < 10 or subquery.student_id is null)
group by register.student_id
```
