1. **создайте новый кластер PostgresSQL 14**  
   ```
   apt install postgresql-14 -y
   ```
2. **зайдите в созданный кластер под пользователем postgres**  
   ```
   sudo -u postgres psql
   ```
3. **создайте новую базу данных testdb**  
   ```
   create database testdb;
   ```
4. **зайдите в созданную базу данных под пользователем postgres**  
   ```
   \c testdb
   ```
5. **создайте новую схему testnm**  
   ```
   create schema testnm;
   ```
6. **создайте новую таблицу t1 с одной колонкой c1 типа integer**
   ```
   create table t1(c1 integer);
   ```
7. **вставьте строку со значением c1=1**
   ```
   insert into t1(c1) values (1);
   ```
8. **создайте новую роль readonly**
   ```
   create role readonly;
   ```
9. **дайте новой роли право на подключение к базе данных testdb**
   ```
   grant CONNECT ON DATABASE testdb TO readonly;
   ```
10. **дайте новой роли право на использование схемы testnm**  
    ```
    grant USAGE ON SCHEMA testnm TO readonly ;  
    ```
11. **дайте новой роли право на select для всех таблиц схемы testnm**
    ```
    grant SELECT ON ALL tables in schema testnm TO readonly;
    ```
12. **создайте пользователя testread с паролем test123**
    ```
    create user testread with password 'test123';
    \q
    nano /etc/postgresql/14/main/pg_hba.conf
    \\добавил
    local   all             testread                                scram-sha-256
    psql -c 'select pg_reload_conf ();'
    psql -c 'select pg_hba_file_rules ();' \\проверяем что всё без ошибок
    ```
13. **дайте роль readonly пользователю testread**
    ```
    grant readonly TO testread;
    ```
14. **зайдите под пользователем testread в базу данных testdb**
    ```
    psql -U testread -d testdb
    ```
15. **сделайте select * from t1;**  
    ```
    select * from t1;
    permission denied for table t1
    \dt
         List of relations
    Schema | Name | Type  |  Owner
    -------+------+-------+----------
    public | t1   | table | postgres
    ```
16. **получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже), напишите что именно произошло в тексте домашнего задания**  
   нет, т.к. таблица создана в схеме public, а на неё у пользователя testread прав нет  
17. **у вас есть идеи почему? ведь права то дали?**  
   права дали на схему testnm  
18. **посмотрите на список таблиц**  
   выполнено в п.15  
19. **подсказка в шпаргалке под пунктом 20**  
20. **а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)**  
   потому что не перешли в схему testnm перед созданием тамблицы t1      
21. **вернитесь в базу данных testdb под пользователем postgres**
    ```
    psql
    \c testdb
    ```
22. **удалите таблицу t1**
    ```
    drop table t1;
    ```
23. **создайте ее заново но уже с явным указанием имени схемы testnm**
    ```
    create table testnm.t1(c1 integer);
    ```
24. **вставьте строку со значением c1=1**
    ```
    insert into testnm.t1(c1) values (1);
    ```
25. **зайдите под пользователем testread в базу данных testdb**
    ```
    psql -U testread -d testdb
    ```
26. **сделайте select * from testnm.t1;**
    ```
    select * from testnm.t1;
    ```
27. **получилось?**  
    нет. т.к. таблица создана после выдачи прав на все таблицы  
28. **есть идеи почему? если нет - смотрите шпаргалку**
    в п.27  
29. **как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку**  
    ```
    \q
    psql
    grant SELECT ON ALL tables in schema testnm TO readonly;
    \q
    ```
30. **сделайте select * from testnm.t1;**  
    ```
    psql -U testread -d testdb
    select * from testnm.t1;
    ```
31. **получилось?**  
    да  
32. **ура!**  
33. **теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);**  
    ```
    create table t2(c1 integer); insert into t2 values (2);  
    ```
34. **а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?**  
    потому что мы создаем в public схеме, а там есть доступ по умолчанию у всех кто может туда зайти  
35. **есть идеи как убрать эти права? если нет - смотрите шпаргалку, если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды**  
    ```
    \\из шпоргалки
    \c testdb postgres; 
    REVOKE CREATE on SCHEMA public FROM public; 
    REVOKE ALL on DATABASE testdb FROM public; 
    \c testdb testread;
    ```
36. **теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);**  
    ```
    ERROR:  permission denied for schema public
    ```
37. **расскажите что получилось и почему**  
    ```
    \\это отбирает наследуемые от группы public права на создание обьектов в схеме public в текущей бд для всех пользователей (если они не указаны явно).
    ```
