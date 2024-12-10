Описание задания:  
На основе готовой базы данных примените один из методов секционирования в зависимости от структуры данных.  
https://postgrespro.ru/education/demodb  
  
Шаги выполнения домашнего задания:  
Анализ структуры данных:  
Схема базы данных flights 
![image](https://github.com/user-attachments/assets/b49767f0-e402-4324-b56f-7df4db50e389)

Ознакомьтесь с таблицами базы данных, особенно с таблицами bookings, tickets, ticket_flights, flights, boarding_passes, seats, airports, aircrafts.  
```
\c demo
demo=# \dt+
                                                 List of relations
  Schema  |      Name       | Type  |  Owner   | Persistence | Access method |  Size   |        Description
----------+-----------------+-------+----------+-------------+---------------+---------+---------------------------
 bookings | aircrafts_data  | table | postgres | permanent   | heap          | 16 kB   | Aircrafts (internal data)
 bookings | airports_data   | table | postgres | permanent   | heap          | 56 kB   | Airports (internal data)
 bookings | boarding_passes | table | postgres | permanent   | heap          | 109 MB  | Boarding passes
 bookings | bookings        | table | postgres | permanent   | heap          | 30 MB   | Bookings
 bookings | flights         | table | postgres | permanent   | heap          | 6368 kB | Flights
 bookings | seats           | table | postgres | permanent   | heap          | 96 kB   | Seats
 bookings | ticket_flights  | table | postgres | permanent   | heap          | 154 MB  | Flight segment
 bookings | tickets         | table | postgres | permanent   | heap          | 109 MB  | Tickets
(8 rows)
```
Определите, какие данные в таблице bookings или других таблицах имеют логическую привязку к диапазонам, по которым можно провести секционирование (например, дата бронирования, рейсы). 
```
demo=# \d bookings
                        Table "bookings.bookings"
    Column    |           Type           | Collation | Nullable | Default
--------------+--------------------------+-----------+----------+---------
 book_ref     | character(6)             |           | not null |
 book_date    | timestamp with time zone |           | not null |
 total_amount | numeric(10,2)            |           | not null |
Indexes:
    "bookings_pkey" PRIMARY KEY, btree (book_ref)
Referenced by:
    TABLE "tickets" CONSTRAINT "tickets_book_ref_fkey" FOREIGN KEY (book_ref) REFERENCES bookings(book_ref)

demo=# select count(book_ref) from bookings;
 count
--------
 593433
(1 row)
```
Выбор таблицы для секционирования:  
Основной акцент делается на секционировании таблицы bookings. Но вы можете выбрать и другие таблицы, если видите в этом смысл для оптимизации производительности (например, flights, boarding_passes).
```
выбираем bookings
```
Обоснуйте свой выбор: почему именно эта таблица требует секционирования? Какой тип данных является ключевым для секционирования?  
```
выбрал по рекомендации задания
```
Определение типа секционирования:  
Определитесь с типом секционирования, которое наилучшим образом подходит для ваших данных:  
По диапазону (например, по дате бронирования или дате рейса).  
По списку (например, по пунктам отправления или по номерам рейсов).  
По хэшированию (для равномерного распределения данных).  
```
выберем по хэшу  
```
Создание секционированной таблицы:
```
demo=# CREATE TABLE bookings_p (book_ref character(6),book_date timestamptz,total_amount numeric(10,2),CONSTRAINT bookings_p_pkey PRIMARY KEY (book_ref)) PARTITION BY HASH(book_ref);
CREATE TABLE

demo=# CREATE TABLE bookings_0 PARTITION OF bookings_p FOR VALUES WITH (MODULUS 10, REMAINDER 0);
CREATE TABLE bookings_1 PARTITION OF bookings_p FOR VALUES WITH (MODULUS 10, REMAINDER 1);
CREATE TABLE bookings_2 PARTITION OF bookings_p FOR VALUES WITH (MODULUS 10, REMAINDER 2);
CREATE TABLE bookings_3 PARTITION OF bookings_p FOR VALUES WITH (MODULUS 10, REMAINDER 3);
CREATE TABLE bookings_4 PARTITION OF bookings_p FOR VALUES WITH (MODULUS 10, REMAINDER 4);
CREATE TABLE bookings_5 PARTITION OF bookings_p FOR VALUES WITH (MODULUS 10, REMAINDER 5);
CREATE TABLE bookings_6 PARTITION OF bookings_p FOR VALUES WITH (MODULUS 10, REMAINDER 6);
CREATE TABLE bookings_7 PARTITION OF bookings_p FOR VALUES WITH (MODULUS 10, REMAINDER 7);
CREATE TABLE bookings_8 PARTITION OF bookings_p FOR VALUES WITH (MODULUS 10, REMAINDER 8);
CREATE TABLE bookings_9 PARTITION OF bookings_p FOR VALUES WITH (MODULUS 10, REMAINDER 9);
```
Преобразуйте таблицу в секционированную с выбранным типом секционирования.  
Например, если вы выбрали секционирование по диапазону дат бронирования, создайте секции по месяцам или годам.  
Миграция данных:  
```
demo=# INSERT INTO bookings_p SELECT * FROM bookings;
INSERT 0 593433
```
Перенесите существующие данные из исходной таблицы в секционированную структуру.  
Убедитесь, что все данные правильно распределены по секциям. 
```
demo=# \dt+
                                                       List of relations
  Schema  |      Name       |       Type        |  Owner   | Persistence | Access method |  Size   |        Description
----------+-----------------+-------------------+----------+-------------+---------------+---------+---------------------------
 bookings | aircrafts_data  | table             | postgres | permanent   | heap          | 16 kB   | Aircrafts (internal data)
 bookings | airports_data   | table             | postgres | permanent   | heap          | 56 kB   | Airports (internal data)
 bookings | boarding_passes | table             | postgres | permanent   | heap          | 109 MB  | Boarding passes
 bookings | bookings        | table             | postgres | permanent   | heap          | 30 MB   | Bookings
 bookings | bookings_0      | table             | postgres | permanent   | heap          | 3048 kB |
 bookings | bookings_1      | table             | postgres | permanent   | heap          | 3080 kB |
 bookings | bookings_2      | table             | postgres | permanent   | heap          | 3064 kB |
 bookings | bookings_3      | table             | postgres | permanent   | heap          | 3048 kB |
 bookings | bookings_4      | table             | postgres | permanent   | heap          | 3040 kB |
 bookings | bookings_5      | table             | postgres | permanent   | heap          | 3048 kB |
 bookings | bookings_6      | table             | postgres | permanent   | heap          | 3064 kB |
 bookings | bookings_7      | table             | postgres | permanent   | heap          | 3040 kB |
 bookings | bookings_8      | table             | postgres | permanent   | heap          | 3088 kB |
 bookings | bookings_9      | table             | postgres | permanent   | heap          | 3080 kB |
 bookings | bookings_p      | partitioned table | postgres | permanent   |               | 0 bytes |
 bookings | flights         | table             | postgres | permanent   | heap          | 6368 kB | Flights
 bookings | seats           | table             | postgres | permanent   | heap          | 96 kB   | Seats
 bookings | ticket_flights  | table             | postgres | permanent   | heap          | 154 MB  | Flight segment
 bookings | tickets         | table             | postgres | permanent   | heap          | 109 MB  | Tickets
(19 rows)

30МБ на 10 частей, всё соответсвует (3МБ на 1 часть)
```
Оптимизация запросов:  
Проверьте, как секционирование влияет на производительность запросов. Выполните несколько выборок данных до и после секционирования для оценки времени выполнения. 
```
demo=# explain analyze select * from bookings where book_ref='013006';
                                                       QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------
 Index Scan using bookings_pkey on bookings  (cost=0.42..8.44 rows=1 width=21) (actual time=1.819..1.821 rows=1 loops=1)
   Index Cond: (book_ref = '013006'::bpchar)
 Planning Time: 0.099 ms
 Execution Time: 1.847 ms
(4 rows)

demo=# explain analyze select * from bookings_p where book_ref='013006';
                                                               QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using bookings_9_pkey on bookings_9 bookings_p  (cost=0.29..8.31 rows=1 width=21) (actual time=0.010..0.011 rows=1 loops=1)
   Index Cond: (book_ref = '013006'::bpchar)
 Planning Time: 0.177 ms
 Execution Time: 0.021 ms
(4 rows)

получилось ускорение в 88 раз
```
Оптимизируйте запросы при необходимости (например, добавьте индексы на ключевые столбцы).  
Учитывая скорость запроса не требуется
Тестирование решения:  
Протестируйте секционирование, выполняя несколько запросов к секционированной таблице.
```
demo=# explain analyze select * from bookings_p where book_ref='013001';
                                                               QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using bookings_9_pkey on bookings_9 bookings_p  (cost=0.29..8.31 rows=1 width=21) (actual time=0.014..0.014 rows=0 loops=1)
   Index Cond: (book_ref = '013001'::bpchar)
 Planning Time: 0.070 ms
 Execution Time: 0.026 ms
(4 rows)
demo=# explain analyze select * from bookings_p where book_ref='014001';
                                                               QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using bookings_1_pkey on bookings_1 bookings_p  (cost=0.29..8.31 rows=1 width=21) (actual time=0.009..0.009 rows=0 loops=1)
   Index Cond: (book_ref = '014001'::bpchar)
 Planning Time: 0.177 ms
 Execution Time: 0.020 ms
(4 rows)

```
Проверьте, что операции вставки, обновления и удаления работают корректно.  
```
demo=# insert into bookings_p values ('000001','2000-01-01','100.0');
INSERT 0 1

demo=# select * from bookings_p where book_ref='000001';
 book_ref |       book_date        | total_amount
----------+------------------------+--------------
 000001   | 2000-01-01 00:00:00+00 |       100.00
(1 row)

demo=# explain analyze select * from bookings_p where book_ref='000001';
                                                               QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------------
 Index Scan using bookings_5_pkey on bookings_5 bookings_p  (cost=0.29..8.31 rows=1 width=21) (actual time=0.023..0.024 rows=1 loops=1)
   Index Cond: (book_ref = '000001'::bpchar)
 Planning Time: 0.074 ms
 Execution Time: 0.035 ms
(4 rows)

работает
```
