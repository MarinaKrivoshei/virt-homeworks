# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1.

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

``` html
vagrant@vagrant:~$ docker run --rm --name pg-docker -e POSTGRES_PASSWORD=postgres -ti -p 5432:5432 -v vol_postgres:/var/lib/postgresql/data postgres:13.0
```
Подключитесь к БД PostgreSQL используя `psql`.

``` html
root@c1348828ff6c:/# psql -h localhost -p 5432 -U postgres -W
Password:
psql (13.0 (Debian 13.0-1.pgdg100+1))
Type "help" for help.
```

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
``` html
postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)
```
- подключения к БД

``` html
postgres=# \c
Password for user postgres:
You are now connected to database "postgres" as user "postgres".
```
- вывода списка таблиц

вывода списка таблиц postgres-# \dt - в таблицах пусто

\dtS : параметр S - для системных объектов

``` html
postgres=# \dt
Did not find any relations.
postgres=# \dtS
                    List of relations
   Schema   |          Name           | Type  |  Owner
------------+-------------------------+-------+----------
 pg_catalog | pg_aggregate            | table | postgres
 pg_catalog | pg_am                   | table | postgres
 pg_catalog | pg_amop                 | table | postgres
 pg_catalog | pg_amproc               | table | postgres
 pg_catalog | pg_attrdef              | table | postgres
 pg_catalog | pg_attribute            | table | postgres
 pg_catalog | pg_auth_members         | table | postgres
 pg_catalog | pg_authid               | table | postgres
 ...
```

- вывода описания содержимого таблиц

``` html
 postgres=# \dS+
                                            List of relations
   Schema   |              Name               | Type  |  Owner   | Persistence |    Size    | Description
------------+---------------------------------+-------+----------+-------------+------------+-------------
 pg_catalog | pg_aggregate                    | table | postgres | permanent   | 56 kB      |
 pg_catalog | pg_am                           | table | postgres | permanent   | 40 kB      |
 pg_catalog | pg_amop                         | table | postgres | permanent   | 80 kB      |
 pg_catalog | pg_amproc                       | table | postgres | permanent   | 64 kB      |
 pg_catalog | pg_attrdef                      | table | postgres | permanent   | 8192 bytes |
 pg_catalog | pg_attribute                    | table | postgres | permanent   | 456 kB     |
 pg_catalog | pg_auth_members                 | table | postgres | permanent   | 40 kB      |
 pg_catalog | pg_authid                       | table | postgres | permanent   | 48 kB      |
 pg_catalog | pg_available_extension_versions | view  | postgres | permanent   | 0 bytes    |
 ...
```

- выхода из psql

``` html
postgres=# \q
root@c1348828ff6c:/#
```

## Задача 2

Используя `psql` создайте БД `test_database`.

``` html
postgres=# CREATE DATABASE test_database;
CREATE DATABASE
```

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

``` html
root@14c9fec359d6:/var/lib/postgresql# psql -U postgres -f test_dump_pg.sql test_database
SET
SET
SET
SET
SET
 set_config
------------

(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
COPY 8
 setval
--------
      8
(1 row)

ALTER TABLE
```

Перейдите в управляющую консоль `psql` внутри контейнера.

``` html
postgres=# \c test_database
Password for user postgres:
You are now connected to database "test_database" as user "postgres".
```

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

``` html

test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE

```

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.

``` html
test_database=# select avg_width from pg_stats where tablename='orders';
 avg_width
-----------
         4
        16
         4
(3 rows)
```

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

``` html

test_database=# ALTER TABLE orders RENAME TO orders_1;
ALTER TABLE
test_database=# CREATE TABLE orders (id integer NOT NULL, title character varying(80) NOT NULL, price integer DEFAULT 0) partition by range(price);
CREATE TABLE
test_database=# CREATE TABLE orders_before_499 partition of orders for values from (0) to (499);
CREATE TABLE
test_database=# create table orders_more499 partition of orders for values from (499) to (999999999);
CREATE TABLE
test_database=# insert into orders (id, title, price) select * from orders_1;
INSERT 0 8

```

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

первоначально создать секционированную таблицу.

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

``` html
root@9462747f2e11:/var/lib/postgresql/data# pg_dump -U postgres -d test_database >test_database_dump.sql
```

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?
``` html
title character varying(80) NOT NULL UNIQUE
alter table orders add unique (title, price);
```
Сделать составное ограничение для секционированной таблицы (т.к ограничения уникальности в секционированных таблицах должны включать все столбцы ключа разбиения.), то выглядеть будет
title character varying(80) NOT NULL UNIQUE
alter table orders add unique (title, price);

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
