### 02. SQL и реляционные СУБД. Введение в PostgreSQL 
Выполнял на своей виртуальной машине Ubuntu 22.04, т.к. в приоритете на данный момент on-premise решение.
___
1. поставить PostgreSQL
>psql (PostgreSQL) 15.1 (Ubuntu 15.1-1.pgdg22.04+1)
2. открыть две сессии и перейти в консоль psql под пользователем postgres
>sudo -u postgres psql/sudo su postgres && psql
3. выключить auto commit 
> \set AUTOCOMMIT off
> 
> \echo :AUTOCOMMIT - проверка статуса
4. создать в первой сессии новую таблицу и наполнить ее данными 
>create table colorhexcodes(id serial, color text, hexcode text); insert into colorhexcodes(color, hexcode) values('black', '000000'); insert into colorhexcodes(color, hexcode) values('white', 'ffffff'); commit;

>CREATE TABLE  
INSERT 0 1  
INSERT 0 1  
COMMIT

5. посмотреть текущий уровень изоляции
>show transaction isolation level;   
>read committed

6. начать новую транзакцию в обеих сессиях с текущим уровнем изоляции  

сессия1
>insert into colorhexcodes(color, hexcode) values('red', 'ff0000');

сессия2
>select * from colorhexcodes;

*видите ли вы новую запись и если да то почему?*  

Нет, потому что транзакция не закоммичена.

сессия1
>commit;

*видите ли вы новую запись и если да то почему?*

Да, транзакция завешена и данные появились в таблице. (Во второй сессии открытых транзакций нет, commit об этом скажет при выполнении.)

7. начать новые но уже repeatable read транзации  
>set transaction isolation level repeatable read;

8. добавить новую запись 

сессия1
>insert into colorhexcodes(color, hexcode) values('blue', '0004ff');

сессия2
>select * from colorhexcodes;

В первой сессии транзакция висит без коммита.

сессия1
>commit;

*видите ли вы новую запись и если да то почему?*

Нет, потому что repeateble read на транзакции.

сессия2
>commit;  
>select * from colorhexcodes;
*видите ли вы новую запись и если да то почему?* 

Вижу, потому что repeatable read слетает после завершения транзакции на read commited.
