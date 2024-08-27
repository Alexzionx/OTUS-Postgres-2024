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
   ```
   
   ```
6. 
