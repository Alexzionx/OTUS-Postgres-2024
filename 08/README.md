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
4. Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
   ```
   ```
5. Протестировать заново
   ```
   ```
6. Что изменилось и почему?

   ```
   ```
7. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
   ```
   ```
8. Посмотреть размер файла с таблицей
   ```
   ```
9. 5 раз обновить все строчки и добавить к каждой строчке любой символ
   ```
   ```
10. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
   ```
   ```
11. Подождать некоторое время, проверяя, пришел ли автовакуум
   ```
   ```
12. 5 раз обновить все строчки и добавить к каждой строчке любой символ
   ```
   ```
13. Посмотреть размер файла с таблицей
   ```
   ```
14. Отключить Автовакуум на конкретной таблице
   ```
   ```
15. 10 раз обновить все строчки и добавить к каждой строчке любой символ
   ```
   ```
16. Посмотреть размер файла с таблицей
   ```
   ```
17. Объясните полученный результат
   ```
   ```
18. Не забудьте включить автовакуум)
   ```
   ```
---
Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла.
