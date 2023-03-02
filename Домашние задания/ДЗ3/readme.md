### Работа с базами данных, пользователями и правами
Выполнял на своей виртуальной машине Ubuntu 22.04, т.к. в приоритете на данный момент on-premise решение.
___
1. Установить PostgreSQL 14 и создать базу данных 
>create database colors;

2. Создать схему в дб
>create schema colorsnm;

3. Создать таблицу (в схеме) и заполнсить.
>create table colorsnm.colorhexcodes(id serial, color text, hexcode text); insert into colorsnm.colorhexcodes(color, hexcode) values('black', '000000'); insert into colorsnm.colorhexcodes(color, hexcode) values('white', 'ffffff');

>colors=# select * from colorsnm.colorhexcodes;  
> id | color | hexcode  
>----+-------+---------  
>  1 | black | 000000  
>  2 | white | ffffff  
>(2 rows)  

4. Создать новую роль с правами ro, дать ей права на подключение к colors, использование схемы colorsnm и селект всех таблиц в схеме.

>colors=# create role readonly;  
>CREATE ROLE  
>colors=# grant connect on database colors to readonly;  
>GRANT  
>grant usage on schema colorsnm to readonly;  
>GRANT  
>colors=# grant select on all tables in schema colorsnm to readonly;  
>GRANT  

5. Создать пользователя и связать его с ролью readonly

>colors=# create user colorsread with password '123123';  
>CREATE ROLE  

>colors=# grant readonly to colorsread;  
>GRANT ROLE  

6. Авторизоваться под colorsread и сделать селект.
>colors=> select * from colorsnm.colorhexcodes;  
> id | color | hexcode  
>----+-------+---------  
>  1 | black | 000000  
>  2 | white | ffffff  
>(2 rows)  

Всё ок, потому что первое правило бойцовского клуба - явно указывать схему при создании таблиц. 

7. Создать новую таблицу в схеме colorsnm.
create table colorsnm.colors2(c1 integer);
ERROR:  permission denied for schema colorsnm

Всё ок. 

8. Создать таблицу в схеме public.
>colors=> create table colors2(c1 integer); insert into colors2 values (2);  
>CREATE TABLE  
>INSERT 0 1  

Вот тут пришлось залезть в шпаргалку
>colors=# revoke CREATE on SCHEMA public FROM public;  
>REVOKE  
>colors=# revoke all on DATABASE colors FROM public;  
>REVOKE  

>colors=> create table colors3(c1 integer); insert into colors2 values (2);  
>ERROR:  permission denied for schema public  

Права ушли. 

p.s. попытка удалить пользователя из роли паблик ни к чему не привела, а документация сказала, что роль public *is special*

