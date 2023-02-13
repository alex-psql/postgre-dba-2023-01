### Установка и настройка PostgteSQL в контейнере Docker
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
