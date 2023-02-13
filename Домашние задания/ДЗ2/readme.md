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
5.1 Проверить, что подключение идет с контейнера на контейнер
>sudo docker ps -a  
>CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES  
>68a390f1d73d   postgres:14   "docker-entrypoint.s…"   36 seconds ago   Up 35 seconds   5432/tcp                                    pg-client  
>93eb699d1b00   postgres:14   "docker-entrypoint.s…"   7 minutes ago    Up 7 minutes    0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-server  

>create table colorhexcodes(id serial, color text, hexcode text); insert into colorhexcodes(color, hexcode) values('black', '000000'); insert into colorhexcodes(color, hexcode) values('white', 'ffffff'); commit;

>CREATE TABLE  
INSERT 0 1  
INSERT 0 1  
COMMIT

5. посмотреть текущий уровень изоляции
>show transaction isolation level;   
>read committed

6. начать новую транзакцию в обеих сессиях с текущим уровнем изоляции  
