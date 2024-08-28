1. **создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в ЯО/Virtual Box/докере**  
   Установка Ubuntu 22.04 на Proxmox  
2. **поставьте на нее PostgreSQL через sudo apt**
   ```
   sudo apt install postgresql-14
   ```
3. **проверьте что кластер запущен через sudo -u postgres pg_lsclusters**
   ```
   sudo -u postgres pg_ctlcluster 14 main status
   ```
4. **зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым**
   ```
   sudo -u postgres psql
   postgres=# create table test(c1 text);  
   postgres=# insert into test values('1');  
   \q
   ```
5. **остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop**
   ```
   sudo -u postgres pg_ctlcluster 14 main stop
   ```
6. **создайте новый диск к ВМ размером 10GB добавьте свеже-созданный диск к виртуальной машине проинициализируйте диск согласно инструкции и подмонтировать файловую систему**    
   ```
   mkdir /mnt/data
   sudo mkfs -t ext4 /dev/sdb1
   mount -t ext4 /dev/sdb1 /mnt/data
   ```
7. **сделайте пользователя postgres владельцем /mnt/data**
   ```
   chown -R postgres:postgres /mnt/data/
   ```
9. **перенесите содержимое /var/lib/postgres/14 в /mnt/data**
    ```
    mv /var/lib/postgresql/14/main/ /mnt/data
    ```
10. **попытайтесь запустить кластер**  
    ```
    sudo -u postgres pg_ctlcluster 14 main start
    ```
11. **напишите получилось или нет и почему**
    Error: /var/lib/postgresql/14/main is not accessible or does not exist  
    т.к. нет данные БД были перемещены  
12. **задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его, напишите что и почему поменяли**
      в /etc/postgresql/14/main/postgresql.conf нужно изменить путь data_directory = '/var/lib/postgresql/14/main' на data_directory = '/mnt/data/main'  
13. **попытайтесь запустить кластер**
    ```
    sudo -u postgres pg_ctlcluster 14 main start
    ```  
14. **напишите получилось или нет и почему**  
    да, т.к. теперь путь корректный
15. **зайдите через через psql и проверьте содержимое ранее созданной таблицы**
    ```
    sudo -u postgres psql
    select * from test;
    ```
16. **задание со звездочкой : не удаляя существующий инстанс ВМ сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.**
