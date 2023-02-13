### Установка и настройка PostgteSQL в контейнере Docker
Выполнял на своей виртуальной машине Ubuntu 22.04, т.к. в приоритете на данный момент on-premise решение.
___
1. Установить docker engine
>curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER

2. Создать каталог /var/lib/postgres и выдать права
>sudo mkdir /var/lib/postgres

3. Создать сеть для контейнеров с pg и pg client
>sudo docker network create postgre-net

4. Создать контейнер pg-server
>sudo docker run --name pg-server --network postgre-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

5. Подлкючиться контейнером pg-client к контейнеру pg-server
>sudo docker run -it --rm --network postgre-net --name pg-client postgres:14 psql -h pg-server -U postgres

6. Создать таблицу и заполнить её
>create table colorhexcodes(id serial, color text, hexcode text); insert into colorhexcodes(color, hexcode) values('black', '000000'); insert into colorhexcodes(color, hexcode) values('white', 'ffffff');

7. Подключиться извне
>psql -p 5432 -U postgres -h my.ip.add.ress -d postgres -W

Установлено, свозданные базы и таблицы увидел.

8. Удалить контейнер
> sudo docker stop pg-server  
> sudo docker rm pg-server

Без ключа --volumes каталог, использовавшийся как подключаемый том остается на сервере-хосте.
>sudo ls -la /var/lib/postgres/base  
>total 24  
>drwx------  6 lxd  999 4096 Feb 13 09:33 .  
>drwx------ 19 lxd root 4096 Feb 13 10:02 ..  
>drwx------  2 lxd  999 4096 Feb 13 09:15 1  
>drwx------  2 lxd  999 4096 Feb 13 09:15 13756  
>drwx------  2 lxd  999 4096 Feb 13 09:16 13757  
>drwx------  2 lxd  999 4096 Feb 13 09:49 16384 // моя база данных  

9. Создать контейнер заново и подключиться. 
>sudo docker run --name pg-server --network postgre-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14

>postgres=# \l  
>                                 List of databases  
>   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges  
>-----------+----------+----------+------------+------------+-----------------------  
> dck_db1   | postgres | UTF8     | en_US.utf8 | en_US.utf8 |                       // моя бд
> postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |  
> template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +  
>           |          |          |            |            | postgres=CTc/postgres  
> template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +  
>           |          |          |            |            | postgres=CTc/postgres  
