**Исходные данные**  
создаём таблицу test и заполняем её  
```
create table test as select generate_series as pkey, generate_series::text || (random() * 10)::text as cnumber, (array['Moscow', 'Piter', 'Volgograd', 'Sochi', 'Ekaterinburg'])[floor(random() * 6)] as col1 from generate_series(1, 100000);

```
**1.Создать индекс к какой-либо из таблиц вашей БД**  
ДО
```
hw15=# explain analyse select * from test where pkey = 3;
                                            QUERY PLAN
--------------------------------------------------------------------------------------------------
 Seq Scan on test  (cost=0.00..2069.00 rows=1 width=35) (actual time=0.007..4.957 rows=1 loops=1)
   Filter: (pkey = 3)
   Rows Removed by Filter: 99999
 Planning Time: 0.077 ms
 Execution Time: 4.968 ms
(5 rows)
```
создаём индекс
```
create index test_pkey_idx on test(pkey);
```
Прислать текстом результат команды explain, в которой используется данный индекс  
```
hw15=# explain analyse select * from test where pkey = 2;
                                                     QUERY PLAN
---------------------------------------------------------------------------------------------------------------------
 Index Scan using test_pkey_idx on test  (cost=0.29..8.31 rows=1 width=35) (actual time=0.017..0.018 rows=1 loops=1)
   Index Cond: (pkey = 2)
 Planning Time: 0.187 ms
 Execution Time: 0.032 ms
(4 rows)
```
**2.Реализовать индекс для полнотекстового поиска** 
ДО
```
hw15=# explain analyse select * from test where col1_tsvec @@ to_tsquery('Volgograd');
                                                        QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..19949.18 rows=16590 width=55) (actual time=0.669..161.644 rows=16754 loops=1)
   Workers Planned: 1
   Workers Launched: 1
   ->  Parallel Seq Scan on test  (cost=0.00..17290.18 rows=9759 width=55) (actual time=0.474..149.649 rows=8377 loops=2)
         Filter: (col1_tsvec @@ to_tsquery('Volgograd'::text))
         Rows Removed by Filter: 41623
 Planning Time: 0.068 ms
 Execution Time: 162.527 ms
(8 rows)
```
Создаём колонку и индекс
```
alter table test add column col1_tsvec tsvector;
update test set col1_tsvec=to_tsvector(col1);
hw15=# CREATE INDEX idx_gin ON test USING gin ("col1_tsvec");
```
ПОСЛЕ
```
hw15=# explain analyse select * from test where col1_tsvec @@ to_tsquery('Volgograd');
                                                        QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on test  (cost=160.82..6364.70 rows=16590 width=55) (actual time=1.488..4.275 rows=16754 loops=1)
   Recheck Cond: (col1_tsvec @@ to_tsquery('Volgograd'::text))
   Heap Blocks: exact=1030
   ->  Bitmap Index Scan on idx_gin  (cost=0.00..156.67 rows=16590 width=0) (actual time=1.383..1.383 rows=16754 loops=1)
         Index Cond: (col1_tsvec @@ to_tsquery('Volgograd'::text))
 Planning Time: 0.221 ms
 Execution Time: 4.958 ms
(7 rows)
```
**3.Реализовать индекс на часть таблицы или индекс на поле с функцией**  
ДО
```
hw15=# explain analyse select * from test where col1 = 'Ekaterinburg';
                                                QUERY PLAN
----------------------------------------------------------------------------------------------------------
 Seq Scan on test  (cost=0.00..3099.00 rows=16693 width=55) (actual time=0.354..7.283 rows=16741 loops=1)
   Filter: (col1 = 'Ekaterinburg'::text)
   Rows Removed by Filter: 83259
 Planning Time: 0.051 ms
 Execution Time: 7.958 ms
(5 rows)
```
Создаём индекс
```
create index test_idx1 on test(lower(col1));
```
ПОСЛЕ
```
hw15=# explain analyse select * from test where col1 = 'Ekaterinburg';
                                                QUERY PLAN
----------------------------------------------------------------------------------------------------------
 Seq Scan on test  (cost=0.00..3099.00 rows=16693 width=55) (actual time=0.351..7.301 rows=16741 loops=1)
   Filter: (col1 = 'Ekaterinburg'::text)
   Rows Removed by Filter: 83259
 Planning Time: 0.187 ms
 Execution Time: 7.977 ms
(5 rows)

Ускорения почему то не получили
```
**4.Создать индекс на несколько полей**  
ДО
```
hw15=# explain analyse select * from test where pkey=119 and col1 = 'Piter';
                                            QUERY PLAN
--------------------------------------------------------------------------------------------------
 Seq Scan on test  (cost=0.00..3348.00 rows=1 width=56) (actual time=5.965..5.966 rows=0 loops=1)
   Filter: ((pkey = 119) AND (col1 = 'Piter'::text))
   Rows Removed by Filter: 100000
 Planning Time: 0.058 ms
 Execution Time: 5.978 ms
(5 rows)
```
Создаём индекс
```
hw15=# create index test_idx2 on test(pkey,col1);
```
ПОСЛЕ
```
hw15=# explain analyse select * from test where pkey=119 and col1 = 'Piter';
                                                   QUERY PLAN
-----------------------------------------------------------------------------------------------------------------
 Index Scan using test_idx2 on test  (cost=0.42..8.44 rows=1 width=56) (actual time=0.035..0.035 rows=0 loops=1)
   Index Cond: ((pkey = 119) AND (col1 = 'Piter'::text))
 Planning Time: 0.265 ms
 Execution Time: 0.047 ms
(4 rows)
```
