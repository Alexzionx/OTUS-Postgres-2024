1. **Cоздайте новый кластер PostgresSQL 14**  
   ```
   apt install postgresql-14 -y
   ```
2. **Создать БД для тестов: выполнить pgbench -i postgres**  
   ```
   sudo su - postgres
   psql -c "create database test;"
   pgbench -i postgres
   ```
3. **Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres**  
   ```
   pgbench -c8 -P 6 -T 60 -U postgres postgres
   --
   pgbench (14.13 (Ubuntu 14.13-0ubuntu0.22.04.1))
   starting vacuum...end.
   progress: 6.0 s, 884.7 tps, lat 9.007 ms stddev 2.200
   progress: 12.0 s, 885.0 tps, lat 9.038 ms stddev 1.998
   progress: 18.0 s, 894.3 tps, lat 8.945 ms stddev 2.083
   progress: 24.0 s, 861.3 tps, lat 9.288 ms stddev 9.365
   progress: 30.0 s, 886.1 tps, lat 9.026 ms stddev 3.279
   progress: 36.0 s, 885.7 tps, lat 9.033 ms stddev 2.677
   progress: 42.0 s, 898.0 tps, lat 8.908 ms stddev 2.445
   progress: 48.0 s, 883.2 tps, lat 9.059 ms stddev 1.973
   progress: 54.0 s, 844.5 tps, lat 9.472 ms stddev 9.517
   progress: 60.0 s, 888.3 tps, lat 9.004 ms stddev 3.457
   transaction type: <builtin: TPC-B (sort of)>
   scaling factor: 1
   query mode: simple
   number of clients: 8
   number of threads: 1
   duration: 60 s
   number of transactions actually processed: 52875
   latency average = 9.077 ms
   latency stddev = 4.756 ms
   initial connection time = 19.412 ms
   tps = 881.200374 (without initial connection time)
   ---
   ```
4. **Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла**
   ```
   psql -c "alter system set max_connections = 40"
   psql -c "alter system set shared_buffers = '1GB'"
   psql -c "alter system set effective_cache_size = '3GB'"
   psql -c "alter system set maintenance_work_mem = '512MB'"
   psql -c "alter system set checkpoint_completion_target = 0.9"
   psql -c "alter system set wal_buffers = '16MB'"
   psql -c "alter system set default_statistics_target = 500"
   psql -c "alter system set random_page_cost = 4"
   psql -c "alter system set effective_io_concurrency = 2"
   psql -c "alter system set work_mem = '6553kB'"
   psql -c "alter system set min_wal_size = '4GB'"
   psql -c "alter system set max_wal_size = '16GB'"
   под ROOT - ~$ pg_ctlcluster restart 14 main
   ```
5. **Протестировать заново**
   ```
   pgbench (14.13 (Ubuntu 14.13-0ubuntu0.22.04.1))
   starting vacuum...end.
   progress: 6.0 s, 865.0 tps, lat 9.210 ms stddev 2.279
   progress: 12.0 s, 854.0 tps, lat 9.367 ms stddev 9.884
   progress: 18.0 s, 888.3 tps, lat 9.005 ms stddev 2.609
   progress: 24.0 s, 903.3 tps, lat 8.821 ms stddev 2.366
   progress: 30.0 s, 903.3 tps, lat 8.890 ms stddev 2.554
   progress: 36.0 s, 856.7 tps, lat 9.338 ms stddev 9.501
   progress: 42.0 s, 887.5 tps, lat 9.014 ms stddev 2.725
   progress: 48.0 s, 881.8 tps, lat 9.071 ms stddev 2.670
   progress: 54.0 s, 880.3 tps, lat 9.086 ms stddev 2.965
   progress: 60.0 s, 854.7 tps, lat 9.360 ms stddev 9.428
   transaction type: <builtin: TPC-B (sort of)>
   scaling factor: 1
   query mode: simple
   number of clients: 8
   number of threads: 1
   duration: 60 s
   number of transactions actually processed: 52658
   latency average = 9.114 ms
   latency stddev = 5.641 ms
   initial connection time = 20.447 ms
   tps = 877.571508 (without initial connection time)
   ```
6. **Что изменилось и почему?**
   ```
   немного величилась задержка stddev , в остальном изменений особо нет (в рамках погрешности)
   ```
7. **Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк**
   ```
   \c test
   create table t1(col text);
   INSERT INTO t1(col) SELECT * FROM generate_series(1,1000000);
   ```
8. **Посмотреть размер файла с таблицей**
   ```
   \dt+ t1
   ---
                                     List of relations
   Schema | Name | Type  |  Owner   | Persistence | Access method | Size  | Description
   -------+------+-------+----------+-------------+---------------+-------+-------------
   public | t1   | table | postgres | permanent   | heap          | 35 MB |
   (1 row)
   ---
   ```
9. **5 раз обновить все строчки и добавить к каждой строчке любой символ**
   ```
   test=# update t1 set col=col||'t';
   test=# update t1 set col=col||'q';
   test=# update t1 set col=col||'5';
   test=# update t1 set col=col||'1';
   test=# update t1 set col=col||'l';
   ```
10. **Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум**
   ```
   SELECT relname, n_live_tup, n_dead_tup, last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 't1';  
   relname | n_live_tup | n_dead_tup |        last_autovacuum  
   ---------+------------+------------+-------------------------------  
   t1      |    1000000 |    4999870 | 2024-10-06 12:32:39.754453+00  
   ```
11. **Подождать некоторое время, проверяя, пришел ли автовакуум**
   ```
   relname | n_live_tup | n_dead_tup |        last_autovacuum
   ---------+------------+------------+-------------------------------
   t1      |    1000000 |          0 | 2024-10-06 12:33:40.462274+00
   ```
12. **5 раз обновить все строчки и добавить к каждой строчке любой символ**
   ```
   test=# update t1 set col=col||'t';
   test=# update t1 set col=col||'q';
   test=# update t1 set col=col||'5';
   test=# update t1 set col=col||'1';
   test=# update t1 set col=col||'l';
   ```
13. **Посмотреть размер файла с таблицей**
   ```
   ---
   \dt+ t1
                                   List of relations
   Schema | Name | Type  |  Owner   | Persistence | Access method |  Size  | Description
   --------+------+-------+----------+-------------+---------------+--------+-------------
   public | t1   | table | postgres | permanent   | heap          | 260 MB |
   (1 row)
   ---
   ```
14. **Отключить Автовакуум на конкретной таблице**
   ```
   alter table t1 set (autovacuum_enabled = off);
   ```
15. **10 раз обновить все строчки и добавить к каждой строчке любой символ**
   ```
   ```
16. **Посмотреть размер файла с таблицей**
   ```
   ```
17. **Объясните полученный результат**
   ```
   ```
18. **Не забудьте включить автовакуум)**
   ```
   ```
---
Задание со *:  
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.  
Не забыть вывести номер шага цикла.  
