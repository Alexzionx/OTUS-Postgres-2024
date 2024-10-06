**Настройте выполнение контрольной точки раз в 30 секунд.**
```
select * from pg_settings where name='checkpoint_timeout';\gx
-[ RECORD 1 ]---+---------------------------------------------------------
name            | checkpoint_timeout
setting         | 300
unit            | s
category        | Write-Ahead Log / Checkpoints
short_desc      | Sets the maximum time between automatic WAL checkpoints.
extra_desc      |
context         | sighup
vartype         | integer
source          | default
min_val         | 30
max_val         | 86400
enumvals        |
boot_val        | 300
reset_val       | 300
sourcefile      |
sourceline      |
pending_restart | f

postgres=# alter system set checkpoint_timeout = 30;
ALTER SYSTEM

postgres=# select pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=# select * from pg_settings where name='checkpoint_timeout';\gx
-[ RECORD 1 ]---+---------------------------------------------------------
name            | checkpoint_timeout
setting         | 30
unit            | s
category        | Write-Ahead Log / Checkpoints
short_desc      | Sets the maximum time between automatic WAL checkpoints.
extra_desc      |
context         | sighup
vartype         | integer
source          | configuration file
min_val         | 30
max_val         | 86400
enumvals        |
boot_val        | 300
reset_val       | 30
sourcefile      | /var/lib/postgresql/14/main/postgresql.auto.conf
sourceline      | 15
pending_restart | f

SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn
---------------------------
 2/BDE10E40
(1 row)

SELECT * FROM pg_stat_bgwriter \gx
-[ RECORD 1 ]---------+------------------------------
checkpoints_timed     | 168
checkpoints_req       | 6
checkpoint_write_time | 3213143
checkpoint_sync_time  | 444
buffers_checkpoint    | 250512
buffers_clean         | 0
maxwritten_clean      | 0
buffers_backend       | 368024
buffers_backend_fsync | 0
buffers_alloc         | 380813
stats_reset           | 2024-10-06 11:30:17.083871+00
```
**10 минут c помощью утилиты pgbench подавайте нагрузку.**
```
pgbench -i test
pgbench -P 30 -T 600 -c 10 test
pgbench (14.13 (Ubuntu 14.13-0ubuntu0.22.04.1))
starting vacuum...end.
progress: 30.0 s, 805.4 tps, lat 12.400 ms stddev 8.779
progress: 60.0 s, 806.9 tps, lat 12.392 ms stddev 8.743
progress: 90.0 s, 830.8 tps, lat 12.037 ms stddev 5.609
progress: 120.0 s, 829.0 tps, lat 12.063 ms stddev 5.639
progress: 150.0 s, 805.1 tps, lat 12.420 ms stddev 9.169
progress: 180.0 s, 830.4 tps, lat 12.042 ms stddev 5.612
progress: 210.0 s, 830.4 tps, lat 12.041 ms stddev 5.687
progress: 240.0 s, 808.8 tps, lat 12.363 ms stddev 8.987
progress: 270.0 s, 831.1 tps, lat 12.032 ms stddev 5.700
progress: 300.0 s, 832.1 tps, lat 12.017 ms stddev 5.684
progress: 330.0 s, 834.0 tps, lat 11.990 ms stddev 5.768
progress: 360.0 s, 802.7 tps, lat 12.458 ms stddev 9.109
progress: 390.0 s, 829.9 tps, lat 12.049 ms stddev 5.663
progress: 420.0 s, 832.6 tps, lat 12.010 ms stddev 5.609
progress: 450.0 s, 831.9 tps, lat 12.020 ms stddev 5.795
progress: 480.0 s, 808.6 tps, lat 12.367 ms stddev 8.926
progress: 510.0 s, 831.0 tps, lat 12.033 ms stddev 5.629
progress: 540.0 s, 831.5 tps, lat 12.026 ms stddev 7.211
progress: 570.0 s, 810.8 tps, lat 12.333 ms stddev 7.648
progress: 600.0 s, 834.0 tps, lat 11.990 ms stddev 7.301
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 10
number of threads: 1
duration: 600 s
number of transactions actually processed: 493719
latency average = 12.152 ms
latency stddev = 7.049 ms
initial connection time = 31.379 ms
tps = 822.860770 (without initial connection time)

SELECT * FROM pg_stat_bgwriter \gx
-[ RECORD 1 ]---------+------------------------------
checkpoints_timed     | 189
checkpoints_req       | 6
checkpoint_write_time | 3724512
checkpoint_sync_time  | 633
buffers_checkpoint    | 290712
buffers_clean         | 0
maxwritten_clean      | 0
buffers_backend       | 371268
buffers_backend_fsync | 0
buffers_alloc         | 384049
stats_reset           | 2024-10-06 11:30:17.083871+00

```
**Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.**
```
SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn
---------------------------
 2/DF5F49B8
(1 row)

SELECT '2/DF5F49B8'::pg_lsn - '2/BDE10E40'::pg_lsn wal_size;
 wal_size
-----------
 561920888
(1 row)
```
```
Расчет обьёма на одну точку 561920888/всего точек 
всего точек - 21 (по данным checkpoints_timed до и после) 
561920888/21=26758137 байт (26758137/1024/1024=25,5 МБ)
```
**Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?** 
```
checkpoints_req до и после одинаковый, значит все чекпоинты были согласно расписанию
```
**Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.**  
```
ALTER SYSTEM SET synchronous_commit = off;
select pg_reload_conf();
~$ pgbench -P 30 -T 600 -c 10 test
pgbench (14.13 (Ubuntu 14.13-0ubuntu0.22.04.1))
starting vacuum...end.
progress: 30.0 s, 1872.4 tps, lat 5.334 ms stddev 0.565
progress: 60.0 s, 1893.1 tps, lat 5.282 ms stddev 0.493
progress: 90.0 s, 1827.1 tps, lat 5.473 ms stddev 0.692
progress: 120.0 s, 1844.3 tps, lat 5.422 ms stddev 0.521
progress: 150.0 s, 1788.9 tps, lat 5.590 ms stddev 0.820
progress: 180.0 s, 1832.0 tps, lat 5.458 ms stddev 0.531
progress: 210.0 s, 1784.6 tps, lat 5.603 ms stddev 0.867
progress: 240.0 s, 1833.4 tps, lat 5.454 ms stddev 0.513
progress: 270.0 s, 1770.2 tps, lat 5.649 ms stddev 0.876
progress: 300.0 s, 1826.2 tps, lat 5.476 ms stddev 0.515
progress: 330.0 s, 1778.6 tps, lat 5.622 ms stddev 0.877
progress: 360.0 s, 1827.5 tps, lat 5.472 ms stddev 0.535
progress: 390.0 s, 1776.2 tps, lat 5.630 ms stddev 0.903
progress: 420.0 s, 1823.3 tps, lat 5.484 ms stddev 0.530
progress: 450.0 s, 1767.2 tps, lat 5.658 ms stddev 0.898
progress: 480.0 s, 1820.3 tps, lat 5.493 ms stddev 0.533
progress: 510.0 s, 1763.0 tps, lat 5.672 ms stddev 0.909
progress: 540.0 s, 1821.5 tps, lat 5.490 ms stddev 0.636
progress: 570.0 s, 1757.3 tps, lat 5.690 ms stddev 0.966
progress: 600.0 s, 1811.7 tps, lat 5.519 ms stddev 0.711
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 10
number of threads: 1
duration: 600 s
number of transactions actually processed: 1086570
latency average = 5.521 ms
latency stddev = 0.722 ms
initial connection time = 33.880 ms
tps = 1810.940912 (without initial connection time)
```
```
пропускная способность увеличилась о этом говорит tps, latency average и latency stddev
Это из-за того что завершенная транзакция не ожидает подтверждания записи данных на диск
```
**Создайте новый кластер с включенной контрольной суммой страниц.**  
```
pg_dropcluster 14 main --stop
pg_createcluster 14 main
/usr/lib/postgresql/14/bin/pg_checksums --pgdata /var/lib/postgresql/14/main --enable --progress
pg_ctlcluster 14 main start

postgres=# show data_checksums ;
 data_checksums
----------------
 on
(1 row)
```
**Создайте таблицу.**  
```
CREATE TABLE t1 (col text);
```
**Вставьте несколько значений.**  
```
INSERT INTO t1(col) SELECT * FROM generate_series(1,100);
```
**Выключите кластер.**  
```
SELECT pg_relation_filepath('t1');
pg_relation_filepath
----------------------
base/13761/16384
(1 row)

pg_ctlcluster 14 main stop
```
**Измените пару байт в таблице.**  
```
nano /var/lib/postgresql/14/main/base/13761/16384 \\изменил 1 символ пореди строки
```
**Включите кластер и сделайте выборку из таблицы.**  
```
select * from t1;
WARNING:  page verification failed, calculated checksum 29496 but expected 54893
ERROR:  invalid page in block 0 of relation base/13761/16384
```
**Что и почему произошло?** 
```
ошибка т.к. контрольные суммы не совпадают
```
**как проигнорировать ошибку и продолжить работу?**  
```
alter system set ignore_checksum_failure = on;
select pg_reload_conf();
select * from t1;
ошибка по прежнему показывает информационную строку WARNING:  page verification failed, calculated checksum 29496 but expected 54893
но работу с таблицей не блокирует
можно выполнить vacuum FULL t1;
после этого будет пересоздан файл, чексуммы будут совпадать и можно будет выключить ignore_checksum_failure
```
