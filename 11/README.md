развернуть виртуальную машину любым удобным способом
```
Установка Ubuntu 22.04 на Proxmox
```
поставить на неё PostgreSQL 15 любым способом
```
apt install postgresql-14 -y
```
настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины
нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
```
sudo su - postgres
pgbench -i postgres
pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (14.13 (Ubuntu 14.13-0ubuntu0.22.04.1))
starting vacuum...end.
progress: 6.0 s, 813.2 tps, lat 9.799 ms stddev 3.959
progress: 12.0 s, 885.5 tps, lat 9.034 ms stddev 2.748
progress: 18.0 s, 887.8 tps, lat 9.010 ms stddev 2.724
progress: 24.0 s, 884.8 tps, lat 9.042 ms stddev 2.440
progress: 30.0 s, 837.7 tps, lat 9.548 ms stddev 10.969
progress: 36.0 s, 868.0 tps, lat 9.216 ms stddev 3.814
progress: 42.0 s, 865.5 tps, lat 9.243 ms stddev 3.891
progress: 48.0 s, 867.2 tps, lat 9.224 ms stddev 4.876
progress: 54.0 s, 839.5 tps, lat 9.531 ms stddev 10.358
progress: 60.0 s, 882.7 tps, lat 9.062 ms stddev 2.721
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 60 s
number of transactions actually processed: 51799
latency average = 9.265 ms
latency stddev = 5.651 ms
initial connection time = 19.003 ms
tps = 863.262612 (without initial connection time)

добавляем в настройки (как и указано игнорируем все возможные проблемы)
fsync = off
synchronous_commit = off 
wal_level = minimal
max_wal_senders = 0
checkpoint_timeout = 60min
full_page_writes = off

pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (14.13 (Ubuntu 14.13-0ubuntu0.22.04.1))
starting vacuum...end.
progress: 6.0 s, 2284.6 tps, lat 3.489 ms stddev 0.435
progress: 12.0 s, 2333.2 tps, lat 3.429 ms stddev 0.269
progress: 18.0 s, 2325.5 tps, lat 3.440 ms stddev 0.292
progress: 24.0 s, 2326.0 tps, lat 3.439 ms stddev 0.289
progress: 30.0 s, 2316.0 tps, lat 3.454 ms stddev 0.306
progress: 36.0 s, 2295.0 tps, lat 3.486 ms stddev 0.423
progress: 42.0 s, 2324.2 tps, lat 3.442 ms stddev 0.410
progress: 48.0 s, 2349.8 tps, lat 3.404 ms stddev 0.307
progress: 54.0 s, 2364.7 tps, lat 3.383 ms stddev 0.268
progress: 60.0 s, 2353.2 tps, lat 3.399 ms stddev 0.352
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 60 s
number of transactions actually processed: 139641
latency average = 3.436 ms
latency stddev = 0.345 ms
initial connection time = 18.703 ms
tps = 2327.640024 (without initial connection time)

пропускная способность увеличилась о этом говорит tps, latency average и latency stddev
```
написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему
```
tps = 2327.640024

fsync = off
synchronous_commit = off
wal_level = minimal
max_wal_senders = 0
checkpoint_timeout = 60min
full_page_writes = off

эти параметры больше всех влияют на производительность СУБД в условиях данного тестирования 
```
Задание со *: аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench)
```
не выполнял
```
