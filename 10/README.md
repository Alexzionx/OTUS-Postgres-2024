**Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд.** 
```
select * from pg_settings where name='deadlock_timeout' \gx
-[ RECORD 1 ]---+--------------------------------------------------------------
name            | deadlock_timeout
setting         | 1000
unit            | ms
category        | Lock Management
short_desc      | Sets the time to wait on a lock before checking for deadlock.
extra_desc      |
context         | superuser
vartype         | integer
source          | default
min_val         | 1
max_val         | 2147483647
enumvals        |
boot_val        | 1000
reset_val       | 1000
sourcefile      |
sourceline      |
pending_restart | f

alter system set deadlock_timeout to 200;
select pg_reload_conf();
create database test;
\c test
CREATE TABLE t1 (col text);
INSERT INTO t1(col) SELECT * FROM generate_series(1,100);
```
**Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.**  
```
запускаем 3 терминала и начинаем 2 транзакции
сессия 1--->
begin;
update t1 set col = '10001' where col='1';
сессия 2--->
update t1 set col = '10001' where col='1';
темрминал 3--->
  SELECT blocked_locks.pid     AS blocked_pid,
         blocked_activity.usename  AS blocked_user,
         blocking_locks.pid     AS blocking_pid,
         blocking_activity.usename AS blocking_user,
         blocked_activity.query    AS blocked_statement,
         blocking_activity.query   AS current_statement_in_blocking_process
   FROM  pg_catalog.pg_locks         blocked_locks
    JOIN pg_catalog.pg_stat_activity blocked_activity  ON blocked_activity.pid = blocked_locks.pid
    JOIN pg_catalog.pg_locks         blocking_locks 
        ON blocking_locks.locktype = blocked_locks.locktype
        AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
        AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
        AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
        AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
        AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
        AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
        AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
        AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
        AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
        AND blocking_locks.pid != blocked_locks.pid
    JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
   WHERE NOT blocked_locks.granted;
 blocked_pid | blocked_user | blocking_pid | blocking_user |             blocked_statement              |   current_statement_in_blocking_process
-------------+--------------+--------------+---------------+--------------------------------------------+--------------------------------------------
       21936 | postgres     |        22127 | postgres      | update t1 set col = '10001' where col='1'; | update t1 set col = '10001' where col='1';
(1 row)
```
**Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах.**
```
запускаем 4 терминала и начинаем 3 транзакции
сессия 1--->
begin;
c
сессия 2--->
begin;
update t1 set col = '10001' where col='1';
темрминал 3--->
begin;
update t1 set col = '10001' where col='1';
темрминал 4--->
   SELECT blocked_locks.pid     AS blocked_pid,
         blocked_activity.usename  AS blocked_user,
         blocking_locks.pid     AS blocking_pid,
         blocking_activity.usename AS blocking_user,
         blocked_activity.query    AS blocked_statement,
         blocking_activity.query   AS current_statement_in_blocking_process
   FROM  pg_catalog.pg_locks         blocked_locks
    JOIN pg_catalog.pg_stat_activity blocked_activity  ON blocked_activity.pid = blocked_locks.pid
    JOIN pg_catalog.pg_locks         blocking_locks
        ON blocking_locks.locktype = blocked_locks.locktype
        AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
        AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
        AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
        AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
        AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
        AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
        AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
        AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
        AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
        AND blocking_locks.pid != blocked_locks.pid
    JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
   WHERE NOT blocked_locks.granted;
 blocked_pid | blocked_user | blocking_pid | blocking_user |             blocked_statement              |   current_statement_in_blocking_process
-------------+--------------+--------------+---------------+--------------------------------------------+--------------------------------------------
       21936 | postgres     |        22127 | postgres      | update t1 set col = '10001' where col='1'; | update t1 set col = '10001' where col='1';
       22545 | postgres     |        21936 | postgres      | update t1 set col = '10001' where col='1'; | update t1 set col = '10001' where col='1';
(2 rows)
```
**Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны.**
```
2 блокировки возникшие в сессиях 2 и 3
```
**Пришлите список блокировок и объясните, что значит каждая.**
```
блокировки выше, из них видно pid блокирующено и заблокированного процесса, а так же текст запроса обоих
```
**Воспроизведите взаимоблокировку трех транзакций.**
```
CREATE TABLE t2 (id SERIAL PRIMARY KEY,col TEXT);
insert into t2 (col) values ( '1');
insert into t2 (col) values ( '2');
insert into t2 (col) values ( '3');
select * from t2;
 id | col
----+-----
  1 | 1
  2 | 2
  3 | 3
(3 rows)

```
**Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?**
```
```
**Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?**
```
```
