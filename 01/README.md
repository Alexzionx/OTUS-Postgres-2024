1. **Установка Ubuntu server 22 в VirtualBox**
2. **Установка Postgres**  
   под **root** или с **sudo**
   ```
   apt undate
   apt upgrade -y
   apt install postgresql-14 -y
   ```
3. **заходим под пользователем postgres в psql**
   ```
   sudo su - postgres
   psql
   ```
4. **выключаем автокоммит**
   ```
   postgres=# \set AUTOCOMMIT off
   ```
5. **открываем psql в другой сессии**
6. **в перевой сессии**
   ```
   create table persons(id serial, first_name text, second_name text); 
   insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
   insert into persons(first_name, second_name) values('petr', 'petrov'); 
   commit;
   ```
7. **Посмотреть текущий уровень изоляции**
   ```
   postgres=# show transaction isolation level;
   read committed
   (1 row)
   ```
8. **в первой сессии добавить новую запись**
   ```
   insert into persons(first_name, second_name) values('sergey', 'sergeev');
   ```
9. **сделать select * from persons во второй сессии**
   ```
   postgres=# select * from persons;
   ```
10. **видите ли вы новую запись и если да то почему?**  
    нет, т.к. на этом уровне изоляции пока не выполнен коммит изменения не зафиксированы т.е. для всех других сессий ничего не изменилось.  
11. **завершить первую транзакцию - commit;**
    ```
    postgres=# commit;
    ```
12. **сделать select * from persons; во второй сессии**  
   видите ли вы новую запись и если да то почему?  
   да, т.к. был выполнен коммит и изменения были зафиксированы, т.е. теперь изменения видны другим сессиям.  
13. **завершите транзакцию во второй сессии**
14. **начать новые но уже repeatable read транзации**
    ```
    postgres=# begin;
    BEGIN
    postgres=*# set transaction isolation level repeatable read;
    SET
    postgres=*# show transaction isolation level;
    transaction_isolation
    -----------------------
    repeatable read
    (1 row)
    ```
15. **в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');**
    ```
    postgres=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
    ```
16. **сделать select * from persons во второй сессии**
    ```
    postgres=*# select * from persons;
    ```
17. **видите ли вы новую запись и если да то почему?**
    нет, т.к. не было коммита
18. **завершить первую транзакцию - commit;**
    ```
    postgres=*# commit;
    ```
19. **сделать select from persons во второй сессии**
    ```
    postgres=*# select * from persons;
    ```
20. **видите ли вы новую запись и если да то почему?**  
    нет, т.к. на этом уровне изоляции (repeatable read) данные видно только текущей транзакции, а так тем что начались после коммита
21. **завершить вторую транзакцию**
    ```
    postgres=*# commit;
    ```
22. **сделать select * from persons во второй сессии**
    ```
    postgres=*# select * from persons;
    ```
23. **видите ли вы новую запись и если да то почему?**  
    да, т.к. началась новая транзация
