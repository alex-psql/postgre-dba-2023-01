
### Физический уровень PostgreSQL
Выполнял на своей виртуальной машине Ubuntu 22.04, т.к. в приоритете на данный момент on-premise решение.
___
1. Установить PostgreSQL 14 и проверить, что он запустился
>sudo apt -y install postgresql-14  
>pg_lsclusters

>14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log

2. Создать и заполнить таблицу в бд postgres под пользователем postgres
>create table colorhexcodes(id serial, color text, hexcode text); insert into colorhexcodes(color, hexcode) values('black', '000000'); insert into colorhexcodes(color, hexcode) values('white', 'ffffff');

3. Остановить кластер pg
>sudo -u postgres pg_ctlcluster 14 main stop  
>14  main    5432 down   postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log

4. Создать и подключить диск, выдать права пользователю postgres
>sudo mount /dev/sdb1 /mnt/data  
>sudo chown -R postgres:postgres /mnt/data/

5. Перенести файлы сервера pg14 на подмонтированный раздел
>sudo mv /var/lib/postgresql/14 /mnt/data

6. Запустить кластер
>Error: /var/lib/postgresql/14/main is not accessible or does not exist

Файлы сервера перенесены, а в конфиге остался старый путь.
>sudo nano /etc/postgresql/14/main/postgresql.conf

Меняем путь на новый.

>data_directory = '/mnt/data/14/main'

Стартуем.

>sudo -u postgres pg_ctlcluster 14 main start  
>14  main    5432 online postgres /mnt/data/14/main /var/log/postgresql/postgresql-14-main.log

Запущено успешно с новым путём из конфига.

8. Проверить данные.
>postgres=# select * from colorhexcodes;  
>id | color | hexcode  
>----+-------+---------  
>  1 | black | 000000  
>  2 | white | ffffff  
>(2 rows)

На месте.

9*. В моём случае: создал новую ВМ, установил Ubuntu и PG14, остановил кластер на исходной ВМ, отмонтировал диск, добавил к новой ВМ существующий диск с разделом и фс. Смонтировал к той же папке /mnt/data. Далее остановил кластер 14main, указал /mnt/data/14/main как путь data_directory и запустил кластер. А он и говорит, что кластер принадлежит юзеру с айди 115, которого нет в системе. А значит имя ничего не значит, и нужно сменить владельца на пользователя postgres от текущей вм. Далее всё поднялось, данные на месте. 
