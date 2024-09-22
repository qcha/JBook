# Авиаперелеты для бизнес пользователей

Задачи подобного плана показывают то, как человек умеет работать с `sql`, понимает ли как работают агрегатные функции и `join`-ы.

Данную задачу подсмотрел с доклада на `Youtube` [Евгений Кудашев, ЦИАН Лондон - Cracking the SQL coding interview](https://www.youtube.com/watch?v=y6CWIBKEw_g).

## Условие

Есть история авиаперелетов:

```sql

```

Надо найти всех, кто летал классом комфорт, но еще не летал классом бизнесс

База данных для примера: [Демонстрационная база данных](https://postgrespro.ru/education/demodb)

## Решение

Получим таблицу продаж всех билетов и пользователей:

```sql
create table sales as 
    select 
        tf.ticket_no, 
        ts.passenger_id,
        ts.passenger_name,
        tf.fare_conditions,
        tf.amount
    from ticket_flights as tf
    inner join tickets as ts on tf.ticket_no = ts.ticket_no
```

### В лоб через IN

Первое и самое плохое решение - это в лоб.

Попробуем просто отфильтровать всех, кто летал комфортом и чьего имени нет среди тех, кто летал бизнесс классом:

```sql
select distinct passenger_name
from sales
where fare_conditions = 'Comfort' 
    and passenger_name not in (
        select distinct passenger_name
        from sales
        where fare_conditions = 'Business'
    )
```

Это будет **плохим** решением!

Операция `in` не предназначена для работы с большими множествами, в нашем же случае подзапрос вернет действительно большое множество, что сразу же негативно отразится на работе.

**ВАЖНО**: Не следует писать в `in` вложенные запросы!

### В лоб через exists

Попытка уйти от `in` и вложенного запроса:

```sql
select distinct passenger_name
from sales s1
where fare_conditions = 'Comfort'
    and not exists (
        select distinct passenger_name
        from sales s2
        where fare_conditions = 'Business'
            and s1.passenger_name = s2.passenger_name
    )
```

Решение будет в 2-3 раза быстрее чем через `in`.

В большинстве СУБД это будет неплохим решением!

### В лоб через self-join

Еще одним способом избавится от `in` является `self join`:

```sql
select distinct s1.passenger_name
from sales s1 left join (
    select passenger_name
        from sales
        where fare_conditions = 'Business'
        group by passenger_name
) as s2 
on
    s1.passenger_name = s2.passenger_name
where s2.passenger_name is null
    and fare_conditions = 'Comfort'
```

#### Оптимизация через CTE

Вложенный подзапрос далеко не всегда будет удачно оптимизирован и поэтому лучше вообще его вынести:

```sql
with s2 as (
    select passenger_name
    from sales
    where fare_conditions = 'Business'
    group by passenger_name
)

select distinct s1.passenger_name
from sales s1 
left join s2
on
    s1.passenger_name = s2.passenger_name
where 
    s2.passenger_name is null
        and fare_conditions = 'Comfort'
```

Почему лучше: таким образом мы явно, не надеясь на оптимизации, избежим `nested loop`.

В большинстве СУБД это будет неплохим решением!

### Через case

Решение через условные выражения более лаконичное и простое.

Необходимо сгруппировать пассажиров и просто посчитать их поездки по двум тарифам, отсеяв их через `having`:

```sql
select passenger_name
from sales
group by passenger_name
having sum(case when fare_conditions = 'Business' then 1 else 0 end) = 0
    and sum(case when fare_conditions = 'Comfort' then 1 else 0 end) > 0
```

Или через `count`:

```sql
select passenger_name
from sales
group by passenger_name
having count(case when fare_conditions = 'Business' then 1 end) = 0
    and count(case when fare_conditions = 'Comfort' then 1 end) > 0
```

Однако это достаточно медленное решение для классической СУБД (для колоночных это скорее всего будет хорошим решением), но мы можем его оптимизировать!

#### Добавляем where

Нас интересуют только строки, содержащие тарифы бизнес или комфорт, другие же мы можем легко отбросить, оптимизировав подсчет!

```sql
select passenger_name
from sales
where fare_conditions in ('Business', 'Comfort')
group by passenger_name
having count(case when fare_conditions = 'Business' then 1 end) = 0
    and count(case when fare_conditions = 'Comfort' then 1 end) > 0
```

В большинстве СУБД это будет неплохим решением!
