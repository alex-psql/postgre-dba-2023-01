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


начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
завершить первую транзакцию - commit;
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
завершите транзакцию во второй сессии
начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
завершить первую транзакцию - commit;
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
завершить вторую транзакцию
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему? ДЗ сдаем в виде миниотчета в markdown в гите
