# Заказы в маркетплейсе

Задача с [Youtube](https://youtu.be/kI6N79Q0wU4?t=557).

## Условие

Имеются следующие таблицы:

```sql
create table transactions(
    transactions_ts timestamp,
    user_id int,
    transaction_id int,
    item text
);
  
```

Выведите для каждого пользователя первое наименование, которое он заказал (первое по времени транзакции). Вывод в виде user_id, item.

## Решение

### Решение в лоб

Первое решение в лоб - это просто сделать подзапрос и найти все user_id, min(transactions_ts) и отфильтровать по ним:

```sql
select 
    user_id,
    item
from
    transactions
where
    (user_id, transactions_ts) in (
select
    user_id,
    min(transactions_ts)
from
    transactions
group by 
    user_id
)
```

Кроме того, что это не эффективно и не красиво, так это может привести к ошибке: если у пользователя несколько транзакций в одну и ту же минимальную дату, то запрос вернёт все такие строки. А нам нужна одна.

### Решение distinct on

Вспомним про distinct on, а точнее то, что этот оператор позволяет выбирать то, что останется в disctinct.

Если применить его и указать сортировку по transactions_ts - то мы получим user_id и item первый по времени.

```sql
select distinct on (user_id)
       user_id,
       item
from 
    transactions
order by
    user_id, transactions_ts;
```

### Решение через оконную функцию

```sql
select distinct
       user_id,
       first_value(item) over (partition by user_id order by transactions_ts)
from 
    transactions
```

Здесь `disctinct` нужен, потому что first_value возвращает значение для каждой строки пользователя, а нам нужен только один результат на пользователя.

И именно поэтому такой вариант на больших данных может быть менее производителен и хорош.

Второй вариант - это решить через row_number и фильтр:

```sql
select 
    user_id,
    item
from (
    select
        user_id,
        item,
        row_number() over (
            partition by user_id
            order by transactions_ts
        ) as rn
    from transactions
) t
where rn = 1;
```

Здесь `row_number()` нумерует транзакции каждого пользователя по времени.

Первая транзакция получает rn = 1.

Внешний запрос оставляет только строки с rn = 1, то есть первый заказ каждого пользователя.
