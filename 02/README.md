1. **создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом**  
Установка Ubuntu 22.04 на Proxmox  
2. **поставить на нем Docker Engine**  
    ```
     Add Docker's official GPG key:
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

     Add the repository to Apt sources:
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt install docker-ce -y    
    ```
3. **сделать каталог /var/lib/postgres (т.к. этот каталог у меня уже есть создам /var/lib/postgres_docker)**
   ```
   sudo mkdir /var/lib/postgres_docker
   ```
4. **развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgres_docker**
   ```
   sudo docker run --name postgres-serv -p 5432:5432 -e POSTGRES_PASSWORD=1111 -d -v /var/lib/postgres_docker:/var/lib/postgresql/data postgres:15
   ```
5. **развернуть контейнер с клиентом postgres подключится из контейнера с клиентом к контейнеру с сервером**
   ```
   sudo docker run -it --rm --name postgres-client postgres:15 psql -h 192.168.1.201 -U postgres
   ```
6. **сделаем таблицу с парой строк**
   ```
   create table persons(id serial, first_name text, second_name text); 
   insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
   insert into persons(first_name, second_name) values('petr', 'petrov'); 
   ``` 
7. **подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов ЯО/места установки докера**
   ```
   sudo psql -U postgres -h 192.168.1.201
   ```
8. **удалить контейнер с сервером**
   ```
   docker stop postgres-serv
   docker rm postgres-serv
   ```  
9. **создать его заново** 
   ```
   sudo docker run --name postgres-serv -p 5432:5432 -e POSTGRES_PASSWORD=1111 -d -v /var/lib/postgres_docker:/var/lib/postgresql/data postgres:15
   ```  
10. **подключится снова из контейнера с клиентом к контейнеру с сервером**  
    ```
    sudo psql -U postgres -h 192.168.1.201
    ```  
11. **проверить, что данные остались на месте**  
    ```
    select * from persons;
    данные на месте
    ```
