# Объединение нескольких таблиц

Задача с [Youtube](https://www.youtube.com/watch?v=AIKAffTTAy4).

## Условие

Имеются таблицы:

```sql
create table company(
    id integer primary key,
    name text
);

create table passenger(
    id integer primary key,
    name text
);

create table pass_in_trip(
    id              integer primary key,
    trip_id         integer,
    passenger_id    integer,
    place           text,

    foreign key (passenger_id) references passenger (id)
);

create table trip(
  id            integer primary key,
  company_id    integer,
  plane         text,
  town_from     text,
  town_to       text,
  time_out      timestamp,
  time_in       timestamp,
  
  foreign key (company_id) references company (id),
  foreign key (id) references pass_in_trip (id)
);

 
insert into company(id, name) values (1, 'Big Company');
insert into company(id, name) values (2, 'Small Company');

insert into passenger(id, name) values (1, 'Bruce Wullis');
insert into passenger(id, name) values (2, 'Aleksandr Kuchuk');

insert into pass_in_trip(id, trip_id, passenger_id, place) values (1, 1, 1, 'Bagama');
insert into pass_in_trip(id, trip_id, passenger_id, place) values (2, 2, 1, 'Tokio');
insert into pass_in_trip(id, trip_id, passenger_id, place) values (3, 3, 2, 'Irkutsk');

insert into trip(id, company_id, plane, town_from, town_to) values (1, 1, 'Airbus', 'New York', 'Town2');
insert into trip(id, company_id, plane, town_from, town_to) values (2, 1, 'Airbus', 'Town2', 'New York');
insert into trip(id, company_id, plane, town_from, town_to) values (3, 2, 'Airbus Small', 'Moscow', 'Irkutsk');

```

Напишите запрос, чтобы найти в какие города летал 'Bruce Wullis'.

## Решение

По связям таблиц можно понять: чтобы найти все города, куда летал Брюс Уиллис, надо связать три таблицы: пассажиров (где мы найдем идентификатор Брюса), `pass_in_trip`, где данные о полетах и пассажирах, и `trip`, где уже подробные данные о перелетах.

```sql
select
    t.town_to
from 
    passenger p
inner join
    pass_in_trip pit
on
    p.id = pit.passenger_id
inner join
    trip t
on 
    t.id = pit.trip_id    
where 
    p.name = 'Bruce Wullis'
```
