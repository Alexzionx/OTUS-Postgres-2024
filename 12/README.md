Создаем ВМ/докер c ПГ.
```
Установка Ubuntu 22.04 на Proxmox
apt install postgresql-14 -y
```
Создаем БД, схему и в ней таблицу.
```
sudo su - postgres
psql -c "create database test;"
```
Заполним таблицы автосгенерированными 100 записями.
```
\c test
create table t1(col text);
INSERT INTO t1(col) SELECT * FROM generate_series(1,100);
```
Под линукс пользователем Postgres создадим каталог для бэкапов
```
mkdir /var/lib/postgresql/backups
```
Сделаем логический бэкап используя утилиту COPY
```
\copy t1 to '/var/lib/postgresql/backups/t1_copy.sql';
```
Восстановим в 2 таблицу данные из бэкапа.
```
create table t2(col text);
\copy t2 from '/var/lib/postgresql/backups/t1_copy.sql'
```
Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц
```
pg_dump -d test -Fc -Z9 -f /var/lib/postgresql/backups/dump_db_test.dump
```
Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!
```
create database test2;
pg_restore -d test2 -t t2 /var/lib/postgresql/backups/dump_db_test.dump
```
