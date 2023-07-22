# Студенты и оценки



## Условие

Имеются следующие таблицы:

```sql

create table register
(
    student_id    integer,
    mark          integer
);
```

Напишите запрос, который выведет количество пятерок у студентов, у которых количество троек меньше десяти.

## Решение

```sql
select student_id
from (
    select student_id, count(case when mark=3 then 1 end) cnt3, count(case when mark=5 then 1 end) cnt5
    from register
    group by student_id
) tmp
where cnt3 < 10
```

Либо:

```sql
select t.student_id, count(case when t.mark = 5 then 1 end) as cnt5
from register t 
where (t.mark = 3 or t.mark = 5)
group by t.student_id
having count(case when t.mark = 3 then 1 end) < 10;
```


```sql
SELECT student_id, SUM(count_of_5) 
FROM (
    SELECT student_id, COUNT(CASE WHEN mark=5 THEN 1 END) count_of_5 
    FROM register 
    GROUP BY student_ID 
    HAVING COUNT(CASE WHEN mark=3 THEN 1 END) < 10
    )
```

```sql
SELECT register.student_id, COUNT(mark)
FROM register LEFT JOIN (
    SELECT student_id, COUNT(mark) AS third
    FROM register 
    WHERE (mark = 3) 
    GROUP BY student_id 
    HAVING third < 10) AS sub
```


