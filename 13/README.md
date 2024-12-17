Устанавливаем на 2 хоста postgres  
```
1. 192.168.1.201
2. 192.168.1.202
```
На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.
На 1 ВМ - 
```
create database testdb;
\c testdb
create table test(col text);
create table test2(col text);
CREATE USER subs LOGIN REPLICATION PASSWORD 'pass1234';
grant all ON test to subs;
grant all ON test2 to subs;

добавляем в конец файла nano /etc/postgresql/14/main/pg_hba.conf строку host testdb subs 192.168.1.202/32 scram-sha-256
echo 'host testdb subs 192.168.1.202/32 scram-sha-256' >> /etc/postgresql/14/main/pg_hba.conf
в /etc/postgresql/14/main/postgresql.conf ставим listen_addresses = '0.0.0.0'
systemctl restart postgresql.service
```
На 2 ВМ - 
```
create database testdb2;
\c testdb2
create table test(col text);
create table test2(col text);
CREATE USER subs LOGIN REPLICATION PASSWORD 'pass1234';
grant all ON test to subs;
grant all ON test2 to subs;

добавляем в конец файла nano /etc/postgresql/14/main/pg_hba.conf строку host testdb2 subs 192.168.1.201/32 scram-sha-256
echo 'host testdb subs 192.168.1.201/32 scram-sha-256' >> /etc/postgresql/14/main/pg_hba.conf
в /etc/postgresql/14/main/postgresql.conf ставим listen_addresses = '0.0.0.0'
systemctl restart postgresql.service
```
Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.
На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.
Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.
На 1 ВМ - 
```
CREATE PUBLICATION test FOR TABLE test;
```
На 2 ВМ - 
```
CREATE PUBLICATION test2 FOR TABLE test2;
```
На 1 ВМ - 
```
CREATE SUBSCRIPTION test2 CONNECTION 'host=192.168.1.202 port=5432 user=subs dbname=testdb2 password=pass1234' PUBLICATION test2;
```
На 2 ВМ - 
```
CREATE SUBSCRIPTION test CONNECTION 'host=192.168.1.201 port=5432 user=subs dbname=testdb password=pass1234' PUBLICATION test;
```
На 1 ВМ - 
```
INSERT INTO test(col) SELECT * FROM generate_series(1,5);
```
На 2 ВМ - 
```
INSERT INTO test2(col) SELECT * FROM generate_series(1,10);
```
На 1 ВМ - 
```
select * from test2;
 col
-----
 1
 2
 3
 4
 5
 6
 7
 8
 9
 10
(10 rows)
```
На 2 ВМ - 
```
select * from test;
 col
-----
 1
 2
 3
 4
 5
(5 rows)
```

3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).

хост 3 - 192.168.1.202
```
create database testdb;
\c testdb
create table test(col text);
create table test2(col text);
CREATE USER subs LOGIN REPLICATION PASSWORD 'pass1234';
grant all ON test to subs;
grant all ON test2 to subs;

в /etc/postgresql/14/main/postgresql.conf ставим listen_addresses = '0.0.0.0'
systemctl restart postgresql.service
```
На 1 и 2 хост 
```
добавляем в hba host testdb subs 192.168.1.203/32 scram-sha-256
и делаем
psql -c "select pg_reload_conf()"
```
```
CREATE SUBSCRIPTION test23 CONNECTION 'host=192.168.1.202 port=5432 user=subs dbname=testdb2 password=pass1234' PUBLICATION test2;
CREATE SUBSCRIPTION test3 CONNECTION 'host=192.168.1.201 port=5432 user=subs dbname=testdb password=pass1234' PUBLICATION test;
```

```
select * from test2;
 col
-----
 1
 2
 3
 4
 5
 6
 7
 8
 9
 10
(10 rows)

select * from test;
 col
-----
 1
 2
 3
 4
 5
(5 rows)

```
